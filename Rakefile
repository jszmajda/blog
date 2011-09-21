task :default => [:build, :deploy]

task :build do
  puts "building"
  `jekyll`
end

task :deploy do
  puts "deploying"
  `rsync -uav --delete -e ssh _site/* josh@loki.ws:~/blog/`
end
