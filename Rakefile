require 'rubygems'
require 'rake'
require 'rdoc'
require 'date'
require 'yaml'
require 'tmpdir'
require 'jekyll'
require "bundler/setup"
require "stringex"

# This will be configured for you when you run config_deploy
deploy_branch = "master"
source_branch = "source"

posts_dir       = "_posts"    # directory for blog files
new_post_ext    = "markdown"  # default new post file extension when using the new_post task
new_page_ext    = "markdown"  # default new page file extension when using the new_page task
deploy_dir      = "_deploy"   # deploy directory (for Github pages deployment)
public_dir      = "_site"    # compiled site directory

# default deploy action
deploy_default = "push"



task :default => :publish


desc "compile and run the site"
task :default do
  pids = [
    spawn("jekyll server -w"),
    spawn("scss --watch _assets:assets"),
    spawn("coffee -b -w -o assets -c _assets/*.coffee")
  ]

  trap "INT" do
    Process.kill "INT", *pids
    exit 1
  end

  loop do
    sleep 1
  end
end


desc "Generate blog files"
task :generate do
  system "git checkout source"
  Jekyll::Site.new(Jekyll.configuration({
    "source"      => ".",
    "destination" => "_site"
  })).process
end


task :check_git do
  unless git_clean?
    puts "Dirty repo - commit or discard your changes and run deploy again"
    exit 1
  end
end

desc "Deploy to remote origin"
task :deploy => [:check_git] do
  message = "Site updated at #{Time.now.utc}"

  system "jekyll"
  system "git checkout \"#{deploy_branch}\""
  system "cp -r _site/* . && rm -rf _site/ && touch .nojekyll"

  unless git_clean?
    system "git add . && git commit -m \"#{message}\""
    system "git push origin \"#{deploy_branch}\""
    puts "Pushed to origin with commit message: #{message}"
  else
    puts "No changes to deploy - canceled"
  end

  system "git checkout \"#{source_branch}\""
end


# usage rake new_post[my-new-post] or rake new_post['my new post'] or rake new_post (defaults to "new-post")
desc "Begin a new post in #{posts_dir}"
task :new_post, :title do |t, args|
  if args.title
    title = args.title
  else
    title = get_stdin("Enter a title for your post: ")
  end
  mkdir_p "#{posts_dir}"
  filename = "#{posts_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title.to_url}.#{new_post_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  puts "Creating new post: #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: \"#{title.gsub(/&/,'&amp;')}\""
    post.puts "date: #{Time.now.strftime('%Y-%m-%d %H:%M')}"
    post.puts "comments: true"
    post.puts "categories: "
    post.puts "---"
  end
end


def get_stdin(message)
  print message
  STDIN.gets.chomp
end

def git_clean?
  git_state = `git status 2> /dev/null | tail -n1`
  clean = (git_state =~ /working directory clean/)
end