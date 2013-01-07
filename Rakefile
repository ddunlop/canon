require 'ci/reporter/rake/rspec'
require 'compass'
require 'rspec/core/rake_task'
require 'sprockets'
require 'sprockets/sass'
require File.expand_path('../lib/canon', __FILE__)
require File.expand_path('../lib/tasks/log', __FILE__)

ENV['CI_REPORTS'] = Canon.build_path

if Canon.environment == 'test'
  Rake::Task['ci:setup:rspec'].invoke
end

desc 'Compile all assets'
task :compile => 'clean' do
  log('Compiling assets') do
    FileUtils.mkdir(Canon.build_path)
    File.write(File.join(Canon.build_path, 'canon.js'), Canon.sprockets['canon.js'])
    File.write(File.join(Canon.build_path, 'canon.css'), Canon.sprockets['canon.css'])
  end

  log('Minifying assets') do
    Canon.sprockets.css_compressor = :yui
    Canon.sprockets.js_compressor = :uglifier
    File.write(File.join(Canon.build_path, 'canon.min.js'), Canon.sprockets['canon.js'])
    File.write(File.join(Canon.build_path, 'canon.min.css'), Canon.sprockets['canon.css'])
  end

  log('Copying images') do
    FileList[Canon.images_path + '/*.png'].each do |image|
      FileUtils.copy(image, Canon.build_path)
    end
  end
end

desc 'Clean build output'
task :clean do
  log('Cleaning all build output') do
    FileUtils.remove_dir(Canon.build_path, true)
  end
end

desc 'Lint javascripts and stylesheets'
task :lint => ['lint:stylesheets', 'lint:javascripts']

namespace :lint do
  desc 'Lint javascripts with JSHint'
  task :javascripts do
    log('Linting javascripts') do
      jshint_command = 'node_modules/.bin/jshint lib/canon/javascripts/'
      if Canon.environment == 'test'
        jshint_command += ' --checkstyle-reporter > ' + Canon.build_path + '/jshint.xml'
      end

      system(jshint_command)
    end
  end

  desc 'Lint stylesheets with CSS Lint'
  task :stylesheets => 'compile' do
    log('Linting stylesheets') do
      csslint_command = 'node_modules/.bin/csslint build/canon.css --quiet --ignore="unique-headings"'
      if Canon.environment == 'test'
        csslint_command += ' --format=checkstyle-xml > ' + Canon.build_path + '/csslint.xml'
      end

      system(csslint_command)
    end
  end
end

namespace :spec do
  desc 'Run functional tests'
  RSpec::Core::RakeTask.new(:functional) do |t|
    t.pattern = 'spec/functional/**/*_spec.rb'
  end

  desc 'Run screenshot tests'
  RSpec::Core::RakeTask.new(:screenshot) do |t|
    t.pattern = 'spec/screenshot/**/*_spec.rb'
  end

  desc 'Run unit tests'
  task :unit do
    url = 'http://0.0.0.0:3000/test'
    reporter = Canon.environment == 'test' ? 'xunit' : 'dot'
    mocha_command = "node_modules/.bin/mocha-phantomjs --reporter #{reporter} #{url}"

    if Canon.environment == 'test'
      mocha_command += ' > ' + Canon.build_path + '/unit.xml'
    end

    system(mocha_command)
  end
end

task :default => ['compile', 'lint', 'spec:unit', 'spec:functional', 'spec:screenshot']