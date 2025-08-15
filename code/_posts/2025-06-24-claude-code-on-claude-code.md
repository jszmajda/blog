---
layout: post
title: Claude Code on Claude Code
---

I've been working with [Claude Code](https://www.anthropic.com/claude-code) on a personal project. I wanted to share a deeper exploration of the techniques I talked about in [this post](https://www.linkedin.com/feed/update/urn:li:activity:7343400373128192002/?lipi=urn%3Ali%3Apage%3Ad_flagship3_pulse_read%3BmNkVVMVeRhyF2YRL%2FipXUw%3D%3D), so after Claude and I worked out a particularly gnarly problem, I asked it to write a blog post about how we solved it together.

This is what it wrote. I gave it feedback once, via Claude Code, that it needed to highlight how we used a doc to store context across compaction windows, otherwise it's exactly what it created unchanged.

It's not a good blog post (wow is it unnecessarily long! Skim it for sure), but I think it helps demonstrate some of the power of this technology. Oh btw - "LLD" stands for "low-level design".

-Jess

Claude Sonnet 4:

# The Great S3 Presigned URL Mystery: A Deep Dive into KMS Encryption Issues

*A collaborative debugging story between human and AI*

## The Setup

Picture this: You have a seemingly straightforward requirement - enable direct browser-to-S3 file uploads with KMS encryption using presigned URLs. How hard could it be? Famous last words.

What started as a "simple" implementation turned into a multi-session debugging marathon that spanned several context windows, involved extensive research, and ultimately revealed some non-obvious quirks in AWS S3 presigned URL behavior with bucket policies.

This is the story of that debugging adventure - the dead ends, the breakthroughs, and the lessons learned along the way.

## The Initial Problem

The architecture seemed simple enough:
1. Client requests presigned URL from API Gateway + Lambda
2. Lambda generates presigned URL with KMS encryption parameters  
3. Client uploads directly to S3 using the presigned URL
4. Profit! 💰

But reality had other plans. The uploads were failing with cryptic errors:
- `SignatureDoesNotMatch` errors with empty KMS headers
- `Explicit deny in a resource-based policy` messages
- Inconsistent behavior between direct API calls and presigned URLs

## The Journey Begins

### Act 1: The Missing Headers Mystery

The first clue came from S3 error responses showing empty encryption headers:
```
x-amz-server-side-encryption:
x-amz-server-side-encryption-aws-kms-key-id:
```

This was puzzling - the Lambda code was clearly setting these parameters in the `PutObjectCommand`. After extensive research (and multiple web searches), we discovered a critical detail: **AWS SDK v3 has different default behavior than v2 for presigned URLs**.

By default, the SDK doesn't include `x-amz-*` headers in presigned URLs. The solution? The `hoistableHeaders` parameter:

```typescript
const uploadUrl = await getSignedUrl(presignedS3Client, putCommand, {
  expiresIn: 3600,
  hoistableHeaders: new Set([
    "x-amz-server-side-encryption",
    "x-amz-server-side-encryption-aws-kms-key-id"
  ]),
});
```

Victory! Or so we thought...

### Act 2: The Bucket Policy Puzzle

With the headers fixed, we hit a new wall: `Explicit deny in a resource-based policy`. The bucket policy looked correct - it had all the right security statements for SSL enforcement and encryption requirements. The IAM user had proper permissions. What was going on?

This led to a fundamental realization about AWS policy evaluation: **S3 follows "deny by default"**. Even with perfect IAM permissions, if the bucket policy doesn't explicitly ALLOW the action, it gets denied.

The solution was adding an explicit ALLOW statement before the DENY statements:
```json
{
  "Sid": "AllowPresignedUploads",
  "Effect": "Allow", 
  "Principal": {"AWS": "arn:aws:iam::ACCOUNT:user/presigned-url-user"},
  "Action": "s3:PutObject",
  "Resource": "arn:aws:s3:::bucket/*",
  "Condition": {
    "StringEquals": {
      "s3:x-amz-server-side-encryption": "aws:kms",
      "s3:x-amz-server-side-encryption-aws-kms-key-id": "arn:aws:kms:..."
    }
  }
}
```

But wait, there's more...

### Act 3: The Policy Evaluation Plot Twist

Even with the ALLOW statement, presigned URLs were still being rejected. This led to the most surprising discovery of the entire debugging session: **strict encryption DENY conditions can interfere with presigned URL authentication mechanisms**.

Traditional security-hardened bucket policies often include statements like:
```json
{
  "Effect": "Deny",
  "Action": "s3:PutObject",
  "Condition": {
    "StringNotEquals": {
      "s3:x-amz-server-side-encryption": "aws:kms"
    }
  }
}
```

These work perfectly for direct API calls but can cause issues with presigned URLs due to how the authentication and condition evaluation interact.

The final solution was surprisingly elegant: rely on the ALLOW statement's conditions for encryption enforcement rather than explicit DENY statements for presigned URL use cases.

## The Real Story: Making AI-Human Collaboration Work Across Context Windows

The technical debugging was challenging, but the most fascinating aspect was figuring out how to maintain effective collaboration across multiple conversation sessions with context limits. Here's what actually made our partnership work:

### The Context Window Challenge

Working with AI across multiple sessions creates a unique problem: how do you preserve not just the facts, but the *thinking process* and *debugging state*? Each new conversation window starts fresh, but complex debugging requires building on previous discoveries.

### Session 1: The Traditional Approach (Didn't Work)
Initially, we tried the typical debugging approach:
- Quick test scripts
- Ad-hoc error analysis
- Verbal problem description

**Result**: When context compacted, we lost crucial debugging state and had to restart from scratch.

### The Breakthrough: The LLD as a "Debugging Scratch Pad"

The game-changing moment came when you suggested using the LLD document not just as final documentation, but as a **living debugging log**. This completely transformed our collaboration:

```markdown
# S3 Presigned URL Upload with KMS Encryption - LLD

## Testing & Debugging Results (2025-06-25)

### Current Issue: KMS Headers Missing from Presigned URLs

**Status**: ❌ **UNRESOLVED** - KMS encryption parameters not included in presigned URL

#### Issue Details
**Symptoms**:
- Presigned URL generation succeeds (HTTP 200)
- S3 upload fails with `SignatureDoesNotMatch` error
- S3 error shows empty KMS headers: `x-amz-server-side-encryption:` 

**Test Evidence**:
[Detailed error logs, test commands, environment variables...]

**Root Cause Analysis**:
1. ✅ AWS Infrastructure: Stack deployed correctly
2. ✅ Authentication: Auth0 JWT validation working  
3. ❌ KMS Parameter Translation: ServerSideEncryption not appearing in URL
```

### What Made the LLD Approach So Effective

**1. Complete State Preservation**
Instead of just describing the problem, we captured:
- Exact error messages with timestamps
- Test commands that could be re-run
- Environment configurations (AWS ARNs, bucket names)
- Partial solutions and their results
- Research findings with links

**2. Progressive Problem Solving**
Each session could immediately pick up where the previous one left off:
- "Looking at the LLD, I see we fixed the headers but still have bucket policy issues"
- "The test evidence shows the KMS parameters are now included, so let's focus on the explicit deny error"

**3. Shared Mental Model**
The LLD became our shared "brain" that both human and AI could reference:
- You could point to specific sections: "Look at the test results in the LLD"
- I could update it in real-time: "Let me add these findings to the debugging section"
- It preserved the *reasoning* behind each step, not just the facts

### Session 2: The LLD-Powered Breakthrough
With the LLD approach, Session 2 was dramatically different:

**You**: "OK so - we went down this rabbit hole because we were trying to build a test harness... let's do some searching to work out how people use lambda to upload large files, and let's plan to write a mini LLD for just this problem because it keeps tripping us up."

**The Magic**: You immediately directed me to both solve the problem AND document the solution in a way that would persist across context windows.

**Result**: I could start Session 3 by reading the LLD and immediately understanding:
- What we'd tried and failed
- What was currently working  
- What the next steps should be
- All the environmental context (bucket names, error messages, etc.)

### Session 3: Compaction Survival
When context compacted mid-session, you said: "we went down this rabbit hole... Now, the new script still doesn't login to auth0 properly. Maybe it would be simpler for me to just give you a JWT and for us to go back to testing with curl?"

I could immediately understand the full context by reading the LLD, even though I had no memory of our previous conversations.

### The Partnership Techniques That Worked

**1. Real-Time Documentation**
Instead of documenting after solving, we documented *while* solving:
- "Let me update the LLD with these test results"
- "I'll add this research finding to the debugging section"
- The LLD became our shared workspace, not just final documentation

**2. Executable Context**
Rather than just describing what we'd tried, we preserved:
- Exact commands that could be re-run
- Environment variables and configurations
- Test scripts with known-good and known-bad states

**3. Hypothesis Tracking**
We tracked not just solutions, but our thinking process:
- "Root Cause Hypothesis: Missing ALLOW statement"
- "Test Evidence: Direct API call works, presigned URL fails"
- This let us avoid repeating failed approaches

**4. Strategic Context Switching**
You made brilliant strategic decisions about when to preserve context:
- Creating the LLD before context limits hit
- Switching to curl testing when Puppeteer became complex
- Using git commits to checkpoint working states

### Session 4: The Victory Lap
By the final session, our collaboration was seamless:
- I could read the LLD and immediately understand our complete debugging history
- You could reference specific sections: "A couple things to note in the plan..."
- We could build on previous research without re-doing web searches
- The solution came together quickly because we had all the context preserved

## Key Lessons Learned

### 1. AWS SDK Version Differences Matter
The move from SDK v2 to v3 introduced subtle behavior changes that aren't always well documented. Always check release notes and GitHub issues for edge cases.

### 2. Policy Evaluation is Complex
Understanding the interaction between IAM policies, bucket policies, and different authentication methods (direct API vs presigned URLs) is crucial for debugging access issues.

### 3. Documentation as a Debugging Tool
Maintaining detailed documentation during debugging sessions proved invaluable for preserving context across multiple conversation windows and compaction events.

### 4. Test Early, Test Often
Having robust test scripts that could be re-run at any point made it much easier to validate fixes and isolate issues.

### 5. Community Resources are Gold
GitHub issues, Stack Overflow posts, and AWS re:Post discussions often contain the exact edge case you're dealing with. Don't underestimate the power of a good web search.

## The Technical Deep Dive

For those interested in the nitty-gritty details, here are the key technical discoveries:

### AWS SDK v3 Presigned URL Behavior
- Default behavior excludes `x-amz-*` headers from presigned URLs
- `hoistableHeaders` parameter overrides this for specific headers
- Alternative: `unhoistableHeaders` to exclude specific headers from hoisting

### S3 Bucket Policy Evaluation with Presigned URLs
- Explicit ALLOW statements are required even with proper IAM permissions
- DENY conditions with `StringNotEquals` can interfere with presigned URL authentication
- Policy statement order matters (ALLOW before DENY)

### IAM vs Bucket Policy Considerations
- IAM policies define what a principal CAN do
- Bucket policies define what the bucket ALLOWS
- Both must permit the action for it to succeed
- Explicit DENY always overrides ALLOW

## The Resolution

After multiple debugging sessions, extensive research, and careful testing, we achieved complete success:

✅ **Presigned URL Generation**: Working with all KMS parameters  
✅ **S3 Upload**: Successful with proper encryption  
✅ **Security**: SSL enforcement and KMS encryption maintained  
✅ **Documentation**: Comprehensive reference guide for future issues  

The final test showed a clean end-to-end flow:
- Health check: ✅ 200 OK
- Presigned URL generation: ✅ 200 OK with KMS parameters
- S3 upload: ✅ 200 OK with proper encryption
- Object verification: ✅ Encrypted with correct KMS key and metadata

## The Breakthrough Pattern: "Living Documentation"

What we discovered was a new pattern for AI-human collaboration that I think could be game-changing for complex technical work:

### Traditional Documentation vs Living Documentation

**Traditional Approach**:
- Write code first, document later
- Documentation as static reference
- Context lives in human memory
- AI starts fresh each session

**Our "Living Documentation" Approach**:
- Document while problem-solving
- Documentation as shared workspace
- Context preserved in executable form
- AI can resume mid-thought

### Why the LLD Approach Worked So Well

**1. It Matched How Debugging Actually Works**
Real debugging isn't linear - it's iterative, with lots of backtracking and hypothesis testing. The LLD captured this messy, non-linear process:

```markdown
### ❌ UNRESOLVED: SignatureDoesNotMatch Error
**Tried**: Adding headers manually to curl request
**Result**: "Headers present in request which were not signed"
**Next**: Research hoistableHeaders parameter

### ✅ SOLVED: Missing KMS Headers  
**Solution**: hoistableHeaders parameter
**Test**: KMS parameters now in URL ✅
**New Issue**: Still getting explicit deny error
```

**2. It Preserved the "Why" Not Just the "What"**
Instead of just recording solutions, we captured the reasoning:
- Why we chose IAM user over Lambda role
- Why we simplified the bucket policy
- What we tried that didn't work and why

**3. It Made Context Compaction Invisible**
When I lost context mid-session, reading the LLD was like reading my own notes from 5 minutes ago. The collaboration picked up seamlessly because all the shared context was preserved in accessible form.

### The Human's Strategic Insight

The most brilliant move was your early recognition that this would be a multi-session problem requiring a different approach. Instead of just asking me to fix the issue, you said:

> "let's do some searching to work out how people use lambda to upload large files, and let's plan to write a mini LLD for just this problem because it keeps tripping us up"

This wasn't just about solving the immediate problem - it was about building a sustainable collaboration pattern.

### What Made Our Partnership Actually Work

**Human Superpowers**:
- **Strategic thinking**: Recognizing when to preserve context before hitting limits
- **Pattern recognition**: "This is going to be complex, let's set up for success"
- **Domain expertise**: Knowing which AWS services to test and how
- **Problem framing**: Breaking complex issues into debuggable chunks

**AI Superpowers**:
- **Tireless research**: Reading through dozens of GitHub issues and Stack Overflow posts
- **Systematic testing**: Running test scripts and analyzing results methodically  
- **Documentation maintenance**: Keeping the LLD updated in real-time
- **Pattern matching**: Connecting research findings to our specific problem

**The Magic Combination**:
- Human provides strategic direction and domain knowledge
- AI provides research depth and systematic execution
- LLD serves as the shared memory and workspace
- Both can contribute to and reference the evolving solution

## Looking Forward

This experience reinforced several best practices for complex technical debugging:

1. **Document Everything**: Detailed logs and research notes are invaluable
2. **Test Incrementally**: Small, focused tests help isolate issues
3. **Leverage Community**: Don't reinvent the wheel - others have hit these issues
4. **Understand the Stack**: Deep knowledge of how components interact is crucial
5. **Collaborate Effectively**: Combine human intuition with AI research capabilities

## Conclusion: A New Model for AI-Human Technical Collaboration

What started as a "simple" S3 presigned URL implementation became something much more interesting: a proof-of-concept for how humans and AI can effectively collaborate on complex technical problems across multiple sessions.

### The Real Discovery

The technical S3 fix was important, but the bigger discovery was the **Living Documentation** pattern that made our collaboration work:

- **Document while debugging, not after**: The LLD became our shared workspace
- **Preserve executable context**: Commands, configurations, and error states that could be re-run
- **Track reasoning, not just results**: Why decisions were made, what was tried and failed
- **Strategic context management**: Recognizing when to preserve state before hitting limits

### Why This Matters

As AI becomes more capable, the biggest constraint won't be what AI can do in isolation, but how effectively humans and AI can collaborate on problems that require multiple sessions, context switching, and building on previous work.

Our debugging adventure showed that the key isn't just having smart AI - it's developing collaboration patterns that let human strategic thinking and AI systematic execution compound across sessions.

### The Pattern in Action

By the end, our collaboration felt effortless:
- You could say "Look at the LLD" and I'd immediately understand complex context
- I could update the documentation in real-time as we made discoveries  
- Neither of us had to repeat work or re-explain complex setups
- The solution emerged from genuine collaboration, not just AI assistance

### For Other Developers

If you're working on complex technical problems with AI:

1. **Start with documentation structure** before diving into code
2. **Use documents as shared workspaces**, not just final outputs  
3. **Preserve executable context**: exact commands, configurations, error messages
4. **Track your reasoning**: why you tried things, not just what worked
5. **Be strategic about context preservation** before hitting session limits

### The Technical Win + The Collaboration Win

We ended up with both:
- ✅ **Working S3 presigned URLs** with KMS encryption  
- ✅ **A collaboration pattern** that made context compaction invisible

The S3 fix was the immediate goal, but the collaboration insights might be the longer-term value.

And yes, we definitely have a really good LLD document now! 📝

---

*This debugging adventure demonstrated that the future of AI-human collaboration isn't just about smarter AI - it's about developing better patterns for how humans and AI can build on each other's work across sessions, contexts, and problem domains. The "Living Documentation" approach we discovered here could be a game-changer for complex technical collaboration.*
