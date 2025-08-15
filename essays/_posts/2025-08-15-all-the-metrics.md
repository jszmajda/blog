---
layout: post
title: Developer Metrics That Matter
---

Leading an engineering team without good metrics is like trying to navigate without a map. You might eventually get where you're going, but you'll waste time, energy, and probably frustrate everyone in the process. Yet many teams resist tracking developer metrics, seeing them as overhead or micromanagement.

The truth is, the right metrics aren't about control - they're about clarity. They help teams understand their own effectiveness and improve their delivery process. Here are the key metrics I've found most valuable for engineering teams, along with why they matter and what goals to set.

### High-Impact Metrics You Can Start Tracking Now

These metrics provide immediate insights into your team's delivery process. If you're using standard project management tools like Jira, you likely already have the data you need to calculate them.

#### Cycle Time
**What it is:** The time from when a ticket is first marked "in progress" to when it's released to users.

**Goal:** 80% of tickets should have a cycle time of 3 days or less.

**Why it matters:** Fast cycle times mean faster value delivery to users and lower risk per change. Small chunks are easier for engineers to reason about and less likely to introduce bugs. Teams that consistently hit this goal ship features weeks faster than those that don't.


<svg width="600" height="200" viewBox="0 0 600 200" xmlns="http://www.w3.org/2000/svg">
  <!-- Timeline background -->
  <rect x="50" y="80" width="500" height="40" fill="#f0f0f0" stroke="#ccc" stroke-width="1" rx="5"/>
  
  <!-- Timeline labels -->
  <text x="50" y="70" text-anchor="start" font-family="Arial, sans-serif" font-size="12" fill="#666">Ticket Created</text>
  <text x="550" y="70" text-anchor="end" font-family="Arial, sans-serif" font-size="12" fill="#666">Released</text>
  
  <!-- Cycle time section -->
  <rect x="150" y="85" width="300" height="30" fill="#4CAF50" opacity="0.7" rx="3"/>
  <text x="300" y="105" text-anchor="middle" font-family="Arial, sans-serif" font-size="12" fill="white" font-weight="bold">Cycle Time</text>
  
  <!-- Phase markers -->
  <line x1="150" y1="80" x2="150" y2="120" stroke="#333" stroke-width="2"/>
  <line x1="450" y1="80" x2="450" y2="120" stroke="#333" stroke-width="2"/>
  
  <!-- Phase labels -->
  <text x="150" y="140" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#666">In Progress</text>
  <text x="450" y="140" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#666">Released</text>
  
  <!-- Goal indicator -->
  <text x="300" y="170" text-anchor="middle" font-family="Arial, sans-serif" font-size="14" fill="#333" font-weight="bold">Goal: ≤ 3 days for 80% of tickets</text>
</svg>

#### Velocity Variance
**What it is:** For each sprint, the variance between this sprint's completed story points versus the previous 3 sprints' average.

**Goal:** ≤ 20% variance sprint to sprint.

**Why it matters:** Predictable velocity enables accurate roadmap planning and stakeholder commitments. High variance signals disruption - whether from unclear requirements, production fires, or scope creep. Consistent teams can plan months ahead; inconsistent teams can barely predict next week.


<svg width="600" height="300" viewBox="0 0 600 300" xmlns="http://www.w3.org/2000/svg">
  <!-- Chart background -->
  <rect x="80" y="40" width="450" height="200" fill="#fafafa" stroke="#ddd" stroke-width="1"/>
  
  <!-- Y-axis -->
  <line x1="80" y1="40" x2="80" y2="240" stroke="#333" stroke-width="2"/>
  <text x="30" y="140" text-anchor="middle" font-family="Arial, sans-serif" font-size="12" fill="#666" transform="rotate(-90, 30, 140)">Story Points</text>
  
  <!-- X-axis -->
  <line x1="80" y1="240" x2="530" y2="240" stroke="#333" stroke-width="2"/>
  <text x="305" y="270" text-anchor="middle" font-family="Arial, sans-serif" font-size="12" fill="#666">Sprint</text>
  
  <!-- Grid lines -->
  <line x1="80" y1="60" x2="530" y2="60" stroke="#eee" stroke-width="1"/>
  <line x1="80" y1="100" x2="530" y2="100" stroke="#eee" stroke-width="1"/>
  <line x1="80" y1="140" x2="530" y2="140" stroke="#eee" stroke-width="1"/>
  <line x1="80" y1="180" x2="530" y2="180" stroke="#eee" stroke-width="1"/>
  <line x1="80" y1="220" x2="530" y2="220" stroke="#eee" stroke-width="1"/>
  
  <!-- Y-axis labels -->
  <text x="75" y="65" text-anchor="end" font-family="Arial, sans-serif" font-size="10" fill="#666">50</text>
  <text x="75" y="105" text-anchor="end" font-family="Arial, sans-serif" font-size="10" fill="#666">40</text>
  <text x="75" y="145" text-anchor="end" font-family="Arial, sans-serif" font-size="10" fill="#666">30</text>
  <text x="75" y="185" text-anchor="end" font-family="Arial, sans-serif" font-size="10" fill="#666">20</text>
  <text x="75" y="225" text-anchor="end" font-family="Arial, sans-serif" font-size="10" fill="#666">10</text>
  
  <!-- Acceptable variance band (24-36 points) -->
  <!-- 24 points = y=164, 36 points = y=116 -->
  <path d="M 160 164 L 530 164 L 530 116 L 160 116 Z" fill="#C8E6C9" opacity="0.6"/>
  
  <!-- 3-sprint rolling average line (at 30 points, y=140) -->
  <line x1="160" y1="140" x2="530" y2="140" stroke="#4CAF50" stroke-width="3"/>
  
  <!-- Upper variance bound (36 points, y=116) -->
  <line x1="160" y1="116" x2="530" y2="116" stroke="#4CAF50" stroke-width="2" stroke-dasharray="3,3"/>
  
  <!-- Lower variance bound (24 points, y=164) -->
  <line x1="160" y1="164" x2="530" y2="164" stroke="#4CAF50" stroke-width="2" stroke-dasharray="3,3"/>
  
  <!-- Sprint velocity points -->
  <!-- Sprint 1: 32 points (y=140 - 8 = 132, good) -->
  <circle cx="160" cy="132" r="6" fill="#2196F3" stroke="white" stroke-width="2"/>
  <text x="160" y="255" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#666">1</text>
  
  <!-- Sprint 2: 28 points (y=140 + 8 = 148, good) -->
  <circle cx="210" cy="148" r="6" fill="#2196F3" stroke="white" stroke-width="2"/>
  <text x="210" y="255" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#666">2</text>
  
  <!-- Sprint 3: 34 points (y=140 - 16 = 124, good) -->
  <circle cx="260" cy="124" r="6" fill="#2196F3" stroke="white" stroke-width="2"/>
  <text x="260" y="255" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#666">3</text>
  
  <!-- Sprint 4: 31 points (y=140 - 4 = 136, good) -->
  <circle cx="310" cy="136" r="6" fill="#2196F3" stroke="white" stroke-width="2"/>
  <text x="310" y="255" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#666">4</text>
  
  <!-- Sprint 5: 29 points (y=140 + 4 = 144, good) -->
  <circle cx="360" cy="144" r="6" fill="#2196F3" stroke="white" stroke-width="2"/>
  <text x="360" y="255" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#666">5</text>
  
  <!-- Sprint 6: 18 points (y=140 + 48 = 188, outside variance - problem!) -->
  <circle cx="410" cy="188" r="8" fill="#FF5722" stroke="white" stroke-width="3"/>
  <text x="410" y="255" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#666">6</text>
  
  <!-- Sprint 7: 42 points (y=140 - 48 = 92, outside variance - problem!) -->
  <circle cx="460" cy="92" r="8" fill="#FF5722" stroke="white" stroke-width="3"/>
  <text x="460" y="255" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#666">7</text>
  
  <!-- Labels -->
  <text x="535" y="121" text-anchor="start" font-family="Arial, sans-serif" font-size="9" fill="#4CAF50">36 pts (+20%)</text>
  <text x="535" y="145" text-anchor="start" font-family="Arial, sans-serif" font-size="9" fill="#4CAF50">30 pts (avg)</text>
  <text x="535" y="169" text-anchor="start" font-family="Arial, sans-serif" font-size="9" fill="#4CAF50">24 pts (-20%)</text>
  
  <!-- Problem indicators -->
  <text x="410" y="203" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" fill="#FF5722" font-weight="bold">Too Low!</text>
  <text x="460" y="82" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" fill="#FF5722" font-weight="bold">Too High!</text>
  
  <!-- Legend -->
  <g transform="translate(90, 50)">
    <rect x="0" y="0" width="12" height="12" fill="#C8E6C9" opacity="0.6"/>
    <text x="18" y="10" font-family="Arial, sans-serif" font-size="9" fill="#666">Acceptable Variance (±20%)</text>
    
    <circle cx="140" cy="6" r="4" fill="#2196F3" stroke="white" stroke-width="1"/>
    <text x="150" y="10" font-family="Arial, sans-serif" font-size="9" fill="#666">Good Velocity</text>
    
    <circle cx="240" cy="6" r="5" fill="#FF5722" stroke="white" stroke-width="2"/>
    <text x="252" y="10" font-family="Arial, sans-serif" font-size="9" fill="#666">High Variance - Investigate</text>
  </g>
</svg>

#### Commit vs. Complete
**What it is:** For each sprint, the ratio of story points you committed to versus story points you actually completed.

**Goal:** Complete within 20% variance of what you committed to.

**Why it matters:** Reliable sprint delivery builds stakeholder trust and team confidence. Chronic under-delivery erodes both and usually indicates estimation problems or hidden work. Teams that consistently hit their commitments get more autonomy and resources.

<svg width="600" height="250" viewBox="0 0 600 250" xmlns="http://www.w3.org/2000/svg">
  <!-- Sprint 1 -->
  <g transform="translate(50, 30)">
    <text x="50" y="15" text-anchor="middle" font-family="Arial, sans-serif" font-size="12" font-weight="bold" fill="#333">Sprint 1</text>
    <rect x="10" y="25" width="80" height="30" fill="#E3F2FD" stroke="#2196F3" stroke-width="2"/>
    <text x="50" y="45" text-anchor="middle" font-family="Arial, sans-serif" font-size="11" fill="#1976D2">Committed: 35</text>
    <rect x="10" y="60" width="80" height="30" fill="#C8E6C9" stroke="#4CAF50" stroke-width="2"/>
    <text x="50" y="80" text-anchor="middle" font-family="Arial, sans-serif" font-size="11" fill="#388E3C">Completed: 34</text>
    <text x="50" y="105" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#4CAF50">✓ 97%</text>
  </g>
  
  <!-- Sprint 2 -->
  <g transform="translate(200, 30)">
    <text x="50" y="15" text-anchor="middle" font-family="Arial, sans-serif" font-size="12" font-weight="bold" fill="#333">Sprint 2</text>
    <rect x="10" y="25" width="80" height="30" fill="#E3F2FD" stroke="#2196F3" stroke-width="2"/>
    <text x="50" y="45" text-anchor="middle" font-family="Arial, sans-serif" font-size="11" fill="#1976D2">Committed: 40</text>
    <rect x="10" y="60" width="80" height="30" fill="#C8E6C9" stroke="#4CAF50" stroke-width="2"/>
    <text x="50" y="80" text-anchor="middle" font-family="Arial, sans-serif" font-size="11" fill="#388E3C">Completed: 38</text>
    <text x="50" y="105" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#4CAF50">✓ 95%</text>
  </g>
  
  <!-- Sprint 3 -->
  <g transform="translate(350, 30)">
    <text x="50" y="15" text-anchor="middle" font-family="Arial, sans-serif" font-size="12" font-weight="bold" fill="#333">Sprint 3</text>
    <rect x="10" y="25" width="80" height="30" fill="#E3F2FD" stroke="#2196F3" stroke-width="2"/>
    <text x="50" y="45" text-anchor="middle" font-family="Arial, sans-serif" font-size="11" fill="#1976D2">Committed: 42</text>
    <rect x="10" y="60" width="80" height="30" fill="#FFCDD2" stroke="#F44336" stroke-width="2"/>
    <text x="50" y="80" text-anchor="middle" font-family="Arial, sans-serif" font-size="11" fill="#D32F2F">Completed: 28</text>
    <text x="50" y="105" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#F44336">⚠ 67%</text>
  </g>
  
  <!-- Legend -->
  <g transform="translate(50, 150)">
    <rect x="0" y="0" width="15" height="15" fill="#E3F2FD" stroke="#2196F3" stroke-width="1"/>
    <text x="20" y="12" font-family="Arial, sans-serif" font-size="10" fill="#666">Committed Points</text>
    
    <rect x="0" y="25" width="15" height="15" fill="#C8E6C9" stroke="#4CAF50" stroke-width="1"/>
    <text x="20" y="37" font-family="Arial, sans-serif" font-size="10" fill="#666">Completed Points (Good)</text>
    
    <rect x="200" y="25" width="15" height="15" fill="#FFCDD2" stroke="#F44336" stroke-width="1"/>
    <text x="220" y="37" font-family="Arial, sans-serif" font-size="10" fill="#666">Completed Points (Needs Attention)</text>
  </g>
  
  <!-- Goal text -->
  <text x="300" y="210" text-anchor="middle" font-family="Arial, sans-serif" font-size="14" fill="#333" font-weight="bold">Goal: Complete within 20% of committed points</text>
</svg>

#### Sprint Churn
**What it is:** For each sprint, the ratio of completed tickets that were among those committed at the start versus the total completed tickets.

**Goal:** ≤ 15% churn generally, though this can vary by product maturity.

**Why it matters:** Low churn shows strong planning discipline and stakeholder alignment. High churn signals reactive work patterns that prevent strategic progress. Some responsiveness is good; constant firefighting kills momentum and team morale.

<svg width="600" height="380" viewBox="0 0 600 380" xmlns="http://www.w3.org/2000/svg">
  <!-- Sprint Start -->
  <g transform="translate(50, 30)">
    <text x="100" y="15" text-anchor="middle" font-family="Arial, sans-serif" font-size="14" font-weight="bold" fill="#333">Sprint Start</text>
    <rect x="20" y="30" width="160" height="120" fill="#f9f9f9" stroke="#ccc" stroke-width="2" rx="8"/>
    <text x="100" y="50" text-anchor="middle" font-family="Arial, sans-serif" font-size="12" fill="#666">Committed Tickets</text>
    
    <!-- Committed tickets -->
    <rect x="30" y="60" width="60" height="20" fill="#2196F3" rx="3"/>
    <text x="60" y="73" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" fill="white">Ticket A</text>
    
    <rect x="110" y="60" width="60" height="20" fill="#2196F3" rx="3"/>
    <text x="140" y="73" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" fill="white">Ticket B</text>
    
    <rect x="30" y="90" width="60" height="20" fill="#2196F3" rx="3"/>
    <text x="60" y="103" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" fill="white">Ticket C</text>
    
    <rect x="110" y="90" width="60" height="20" fill="#2196F3" rx="3"/>
    <text x="140" y="103" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" fill="white">Ticket D</text>
    
    <rect x="30" y="120" width="60" height="20" fill="#2196F3" rx="3"/>
    <text x="60" y="133" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" fill="white">Ticket E</text>
  </g>
  
  <!-- Arrow -->
  <path d="M 230 90 L 270 90" stroke="#666" stroke-width="2" marker-end="url(#arrowhead)" fill="none"/>
  <defs>
    <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
      <polygon points="0 0, 10 3.5, 0 7" fill="#666"/>
    </marker>
  </defs>
  
  <!-- Sprint End -->
  <g transform="translate(300, 30)">
    <text x="100" y="15" text-anchor="middle" font-family="Arial, sans-serif" font-size="14" font-weight="bold" fill="#333">Sprint End</text>
    <rect x="20" y="30" width="160" height="170" fill="#f9f9f9" stroke="#ccc" stroke-width="2" rx="8"/>
    <text x="100" y="50" text-anchor="middle" font-family="Arial, sans-serif" font-size="12" fill="#666">Completed Tickets</text>
    
    <!-- Completed original tickets -->
    <rect x="30" y="60" width="60" height="20" fill="#4CAF50" rx="3"/>
    <text x="60" y="73" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" fill="white">Ticket A</text>
    
    <rect x="110" y="60" width="60" height="20" fill="#4CAF50" rx="3"/>
    <text x="140" y="73" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" fill="white">Ticket C</text>
    
    <rect x="30" y="90" width="60" height="20" fill="#4CAF50" rx="3"/>
    <text x="60" y="103" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" fill="white">Ticket D</text>
    
    <!-- Churn tickets (added during sprint) -->
    <rect x="110" y="90" width="60" height="20" fill="#FF9800" rx="3"/>
    <text x="140" y="103" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" fill="white">Ticket X</text>
    
    <rect x="30" y="120" width="60" height="20" fill="#FF9800" rx="3"/>
    <text x="60" y="133" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" fill="white">Ticket Y</text>
    
    <!-- Moved to next sprint section within the box -->
    <text x="100" y="160" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" fill="#666" font-style="italic">Moved to Next Sprint:</text>
    
    <rect x="40" y="170" width="50" height="18" fill="#FFC107" rx="2"/>
    <text x="65" y="182" text-anchor="middle" font-family="Arial, sans-serif" font-size="8" fill="#333">Ticket B</text>
    
    <rect x="110" y="170" width="50" height="18" fill="#FFC107" rx="2"/>
    <text x="135" y="182" text-anchor="middle" font-family="Arial, sans-serif" font-size="8" fill="#333">Ticket E</text>
  </g>
  
  <!-- Legend (moved down to y=250 for better spacing) -->
  <g transform="translate(50, 250)">
    <rect x="0" y="0" width="12" height="12" fill="#4CAF50" rx="2"/>
    <text x="18" y="10" font-family="Arial, sans-serif" font-size="10" fill="#666">Committed & Completed</text>
    
    <rect x="180" y="0" width="12" height="12" fill="#FF9800" rx="2"/>
    <text x="198" y="10" font-family="Arial, sans-serif" font-size="10" fill="#666">Added During Sprint</text>
    
    <rect x="350" y="0" width="12" height="12" fill="#FFC107" rx="2"/>
    <text x="368" y="10" font-family="Arial, sans-serif" font-size="10" fill="#666">Moved to Next Sprint</text>
  </g>
  
  <!-- Calculation (moved down accordingly) -->
  <g transform="translate(50, 290)">
    <text x="250" y="20" text-anchor="middle" font-family="Arial, sans-serif" font-size="13" fill="#333" font-weight="bold">Churn Calculation:</text>
    <text x="250" y="40" text-anchor="middle" font-family="Arial, sans-serif" font-size="12" fill="#333">2 unplanned tickets ÷ 5 total completed = 40% churn</text>
    <text x="250" y="60" text-anchor="middle" font-family="Arial, sans-serif" font-size="12" fill="#F44336" font-weight="bold">⚠ Above 15% goal - investigate planning process</text>
  </g>
</svg>

### Additional Metrics Worth the Investment

These metrics require more sophisticated tooling, but they provide crucial insights into your team's operational effectiveness:

#### Mean Time to Recover (MTTR)
**What it is:** The average time it takes to fix something broken in production, segmented by severity.

**Why it matters:** Fast recovery minimizes user impact and revenue loss. But MTTR also reveals operational maturity - teams with good monitoring, clear runbooks, and practiced incident response recover 10x faster than those without. High MTTR suggests you're flying blind when things break.

#### Change Failure Rate
**What it is:** The percentage of changes to production that lead to an error, segmented by severity.

**Why it matters:** Low failure rates keep users happy and engineers confident. High rates signal weak development practices - insufficient testing, poor code reviews, or risky deployment processes. Teams with high failure rates spend more time fixing than building.

#### Deployment Frequency
**What it is:** How often you push any code changes to production.

**Why it matters:** Frequent deployment correlates with better engineering practices and faster user value delivery. Daily deployers respond to issues and opportunities in hours; monthly deployers take weeks. Counter-intuitively, frequent deployment reduces risk by making each change smaller and more recoverable.

#### Lead Time
**What it is:** The time from committing to work on an idea to delivering it to production.

**Why it matters:** Short lead times mean faster market response and user feedback cycles. Teams with week-long lead times can pivot quickly; those with month-long lead times ship features that are already obsolete. The best teams balance planning depth with delivery speed.

### Why These Metrics Actually Help

The goal isn't to optimize these numbers in isolation. The goal is to use them to have better conversations about your team's effectiveness. When cycle time increases, you can investigate: Are we taking on work that's too complex? Do we need to break stories down further? When sprint churn increases, you can ask: Are we getting too many urgent requests? Do we need better stakeholder communication about our capacity?

Good metrics help you spot patterns and make improvements. They give you objective data to discuss subjective feelings about how work is going. Most importantly, they help you experiment with process changes and measure whether those changes actually improve outcomes.

### Getting Started

If your team isn't tracking any of these metrics yet, start small. Pick one or two that resonate most with your current challenges and begin measuring them. The act of measurement itself often reveals insights about your process, even before you start optimizing.

Remember: these numbers are tools for improvement, not weapons for judgment. The goal is to help your team deliver better software more predictably, not to create stress or competition. Used well, developer metrics help everyone understand how they can contribute to better outcomes.
