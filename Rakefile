#############################################################################
#
# Modified version of jekyllrb Rakefile
# https://github.com/jekyll/jekyll/blob/master/Rakefile
#
#############################################################################

require 'rake'
require 'date'
require 'yaml'

require 'reduce'
require 'image_optim'
require 'image_optim_pack'

CONFIG = YAML.load(File.read('_config.yml'))
USERNAME = CONFIG["username"]
REPO = CONFIG["repo"]
SOURCE_BRANCH = CONFIG["source_branch"]
DESTINATION_BRANCH = CONFIG["destination_brach"]

def check_destination
  unless Dir.exist? CONFIG["destination"]
    sh "git clone https://$GIT_NAME:$GH_TOKEN@github.com/#{USERNAME}/#{REPO}.git #{CONFIG["destination"]}"
  end
end

namespace :site do
  desc "Generate the site and push changes to remote origin"
  task :deploy do
    # Detect pull request
    if ENV['TRAVIS_PULL_REQUEST'].to_s.to_i > 0
      puts 'Pull request detected. Not proceeding with deploy.'
      exit
    end

    # Configure git if this is run in Travis CI
    if ENV["TRAVIS"]
      sh "git config --global user.name $GIT_NAME"
      sh "git config --global user.email $GIT_EMAIL"
      sh "git config --global push.default simple"
    end

    # Make sure destination folder exists as git repo
    check_destination

    sh "git checkout #{SOURCE_BRANCH}"
    Dir.chdir(CONFIG["destination"]) { sh "git checkout #{DESTINATION_BRANCH}" }

    # Generate the site
    sh "bundle exec jekyll build"

    # Commit and push to github
    sha = `git log`.match(/[a-z0-9]{40}/)[0]
    Dir.chdir(CONFIG["destination"]) do
      # check if there is anything to add and commit, and pushes it
      sh "if [ -n '$(git status)' ]; then
            git add --all .;
            git commit -m 'Updating to #{USERNAME}/#{REPO}@#{sha}.';
            git push --quiet origin #{DESTINATION_BRANCH};
         fi"
      puts "Pushed updated branch #{DESTINATION_BRANCH} to GitHub Pages"
    end
  end
end

# Additional tasks
namespace :resources do
  desc "Optimize website resources"
  task :optimize do
    puts "\n## Compressing static original assets"
    original = 0.0
    compressed = 0
    image_optim = ImageOptim.new
    Dir.glob("assets/images/original/**/*.*") do |file|
      case File.extname(file)
        when ".css", ".gif", ".html", ".jpg", ".jpeg", ".js", ".png", ".xml"
          # Create a copy of the file with lossless compression
          newFilePath = "assets/images/#{File.basename(file)}"
          puts "Processing: #{file} - Creating: #{newFilePath}"
          original += File.size(file).to_f
          min = Reduce.reduce(file)
          File.open(newFilePath, "w") do |f|
            f.write(min)
          end

          # Apply additional compression
          image_optim.optimize_image!(newFilePath)

          newFile = File.open(newFilePath, "r")
          compressed += File.size(newFile)
        else
          puts "Skipping: #{file}"
      end
    end
    puts "Total compression %0.2f\%" % (((original-compressed)/original)*100)
  end
end

# Reference Tasks (do not use these)
# namespace :reference do
#   desc "Minify _site/"
#   task :minify do
#     puts "\n## Compressing static assets"
#     original = 0.0
#     compressed = 0
#     Dir.glob("_site/**/*.*") do |file|
#       case File.extname(file)
#         when ".css", ".gif", ".html", ".jpg", ".jpeg", ".js", ".png", ".xml"
#           puts "Processing: #{file}"
#           original += File.size(file).to_f
#           min = Reduce.reduce(file)
#           File.open(file, "w") do |f|
#             f.write(min)
#           end
#           compressed += File.size(file)
#         else
#           puts "Skipping: #{file}"
#       end
#     end
#     puts "Total compression %0.2f\%" % (((original-compressed)/original)*100)
#   end
# end
