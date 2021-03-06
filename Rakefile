begin
  require 'bundler/setup'
rescue LoadError
  puts 'You must `gem install bundler` and `bundle install` to run rake tasks'
end

Bundler::GemHelper.install_tasks

def copy_react_asset(webpack_file, destination_file)
  full_webpack_path = File.expand_path("../react-builds/build/#{webpack_file}", __FILE__)
  full_destination_path = File.expand_path("../lib/assets/react-source/#{destination_file}", __FILE__)
  FileUtils.cp(full_webpack_path, full_destination_path)
end

# Move to `dirname` and execute `yarn {cmd}`
def yarn_run_in(dirname, cmd)
  Dir.chdir(dirname) do
    `yarn #{cmd}`
  end
end

namespace :react do
  desc "Run the JS build process to put files in the gem source"
  task update: [:install, :build, :copy]

  desc "Build the JS bundles with Webpack"
  task :build do
    yarn_run_in("react-builds", "build")
  end

  desc "Copy browser-ready JS files to the gem's asset paths"
  task :copy do
    environments = ["development", "production"]
    environments.each do |environment|
      # Without addons:
      copy_react_asset("#{environment}/react-browser.js", "#{environment}/react.js")
      copy_react_asset("#{environment}/react-server.js", "#{environment}/react-server.js")

      # With addons:
      copy_react_asset("#{environment}/react-browser-with-addons.js", "#{environment}-with-addons/react.js")
      copy_react_asset("#{environment}/react-server-with-addons.js", "#{environment}-with-addons/react-server.js")
    end
  end

  desc "Install the JavaScript dependencies"
  task :install do
    yarn_run_in("react-builds", "upgrade")
  end
end

namespace :ujs do
  desc "Run the JS build process to put files in the gem source"
  task update: [:install, :build, :copy]

  desc "Install the JavaScript dependencies"
  task :install do
    yarn_run_in("react_ujs", "upgrade")
  end


  desc "Build the JS bundles with Webpack"
  task :build do
    yarn_run_in("react_ujs", "build")
  end

  desc "Copy browser-ready JS files to the gem's asset paths"
  task :copy do
    full_webpack_path = File.expand_path("../react_ujs/dist/react_ujs.js", __FILE__)
    full_destination_path = File.expand_path("../lib/assets/javascripts/react_ujs.js", __FILE__)
    FileUtils.cp(full_webpack_path, full_destination_path)
  end

  desc "Publish the package in ./react_ujs/ to npm as `react_ujs`"
  task publish: :update do
    Dir.chdir("react_ujs") do
      `npm publish`
    end
  end
end

require 'appraisal'
require 'rake/testtask'

Rake::TestTask.new(:test) do |t|
  t.libs << 'lib'
  t.libs << 'test'
  t.pattern = ENV['TEST_PATTERN'] || 'test/**/*_test.rb'
  t.verbose = ENV['TEST_VERBOSE'] == '1'
  t.warning = false
end

task default: :test

task :test_setup do
  Dir.chdir("./test/dummy") do
    `yarn install`
  end
end

task test: :test_setup
