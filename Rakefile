require 'open-uri'
  
ssh_user = "username@server.com" # for rsync deployment
remote_root = "/home/username/sitename/public" # for rsync deployment
WORKING_DIR = "working"
COMPRESSOR_CMD = 'java -jar bin/yuicompressor-2.4.2.jar'


FILES = [
    {:name => "reset", :url => "https://github.com/nathansmith/960-Grid-System/raw/master/code/css/reset.css"},
    {:name => "text", :url => "https://github.com/nathansmith/960-Grid-System/raw/master/code/css/text.css"},
    {:name => "960", :url => "https://github.com/nathansmith/960-Grid-System/raw/master/code/css/960.css"}
  ]
  
  
desc "Install prerequisites"
task :install do

  `gem install haml staticmatic --no-ri --no-rdoc`
  
end
    
  
desc "Create a new site"
task :setup  do
  puts "*** Creating site"
  `staticmatic setup .`
  FileUtils.rm_r "src"
  puts "*** Copying templates"
  FileUtils.cp_r "templates/", "src/"
  Rake::Task["ninesixty:setup"].invoke
  puts "*** Done! Preview with   staticmatic preview ."
  
end

desc "Minify JS and CSS"
task :minify => :build do
  puts "*** Removing existing working dir"
  FileUtils.rm_rf WORKING_DIR
  puts "*** Copying working files"
  FileUtils.cp_r "site/", WORKING_DIR
  minify(WORKING_DIR)
end


desc "preview the site"
task :preview do
  puts "*** Previewing the site"
  system "staticmatic preview ."
end

desc "Builds the site"
task :build => 'styles:clear' do
  puts "*** Building the site"
  system "staticmatic build ."
end

desc "Clears and generates new styles, builds and deploys"
task :deploy => :minify do
  puts "*** Deploying the site via SSH to #{ssh_user}"
  system("rsync -avz --delete #{WORKING_DIR}/ #{ssh_user}:#{remote_root}")
end

namespace :styles do
  desc "Clears the styles"
  task :clear do
    puts "*** Clearing styles"
    system "rm -Rfv site/stylesheets/*"
  end
end


namespace :ninesixty do
  desc "Grab 960.gs files from github"
  task :fetch do
    puts "*** Fetching 960.gs from github...."

  
    FileUtils.rm_rf "tmp"
    FileUtils.mkdir "tmp"
    FILES.each do |cssfile|
      name = cssfile[:name]
      url = cssfile[:url]
      outfile = "tmp/#{name}.css"
      infile = Kernel.open(url)
      File.open(outfile, "w"){|f| f << infile.readlines}
    end
  
  end
  
  desc "Create SASS partials of 960.gs files"
  task :sassify => :fetch do
    FileUtils.mkdir_p "src/stylesheets"
    FILES.each do |cssfile|
       name = cssfile[:name]
       url = cssfile[:url]
       puts "*** Converting #{name}.css to src/stylesheets/_#{name}.sass"
      `sass-cnvert tmp/#{name}.css src/stylesheets/_#{name}.sass`
    end  
    
    FileUtils.rm_rf "tmp"
      
  end
  
  desc "Create base stylesheet with 960 included as a partial. Use KEEP_SEPARATE=true to keep reset, text, and 960 as separate partials"
  task :setup => :sassify do
    puts "*** Creating src/stylesheets/_bass.sass..."
    File.open("src/stylesheets/_base.sass", "w") do |f|
      if ENV["KEEP_SEPARATE"]
        f << "@import reset.sass\n"
        f << "@import text.sass\n"
        f << "@import 960.sass\n"
      else
        compact_to_grid
        f << "@import grid.sass"
      end
    end
    puts "src/stylesheets/_base.sass created."
  end
  
end

# Minify all CSS and JS files found within the working
# directory
def minify(working_dir)
  cssfiles = Dir.glob("#{working_dir}/**/*.css")
  jsfiles = Dir.glob("#{working_dir}/**/*.js")
  files = jsfiles + cssfiles
  
  files.each do |file|
    type = File.extname(file) == ".css" ? "css" : "js"
    newfile = file.gsub(".#{type}", ".new.#{type}")
    puts "minifying #{file}"
    `#{COMPRESSOR_CMD} --type #{type} #{file} > #{newfile}`

    if File.size(newfile) > 0
      FileUtils.cp newfile, file
    else
      puts "Minification for #{file} returned a zero-byte file - discarding."
    end
    FileUtils.rm newfile
  
  end
end


# compacts the 960.gs files into a single grid partial.
def compact_to_grid
  puts "*** Compacting 960.gs to a single file"
  File.open "src/stylesheets/_grid.sass", "w" do |f|
    FILES.each do |cssfile|
      name = cssfile[:name]
      url = cssfile[:url]
      sassfile = "src/stylesheets/_#{name}.sass"
      infile = Kernel.open(sassfile)
      f << infile.readlines
      FileUtils.rm_rf sassfile
    end
  end
end
