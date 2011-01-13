require 'rake'
require 'rspec/core'
require 'rspec/core/rake_task'
require "cucumber/rake/task"
require "yard"
require "city"

desc 'Default: run tests'
task :default => :test

desc "Run specs"
RSpec::Core::RakeTask.new do |t|
  t.pattern = "./spec/**/*_spec.rb" # don't need this, it's default.
  # Put spec opts in a file named .rspec in root
end

desc "Run performance specs"
RSpec::Core::RakeTask.new(:perf) do |t|
  t.pattern = "./performance/**/*_spec.rb"
end

desc "Run features"
Cucumber::Rake::Task.new(:features) do |task|
  task.cucumber_opts = ["features", "-f progress"]
end

desc "Run all reports, specs and features"
task :all do
  puts "\nProfiling report"
  Rake::Task['perf'].invoke
  puts "\nCode Coverage report"
  Rake::Task['coverage'].invoke
  puts "\nFlog results"
  Rake::Task['flog'].invoke
  puts "\nSpec Tests"
  Rake::Task['spec'].invoke
  puts "\nCucumber features"
  Rake::Task['features'].invoke
end

desc "Run tests"
task :test do
  Rake::Task['spec'].invoke
  Rake::Task['features'].invoke
end

desc "Flog the code! (*nix only)"
task :flog do
  system('find lib -name \*.rb | xargs flog')
end

desc "Detailed Flog report! (*nix only)"
task :flog_detail do
  system('find lib -name \*.rb | xargs flog -d')
end

desc "Generate code coverage"
RSpec::Core::RakeTask.new(:coverage) do |t|
  t.pattern = "./spec/**/*_spec.rb" # don't need this, it's default.
  t.rcov = true
  t.rcov_opts = ['--exclude', 'spec', '--exclude', 'gems', '-T']
end


YARD::Rake::CitydocTask.new do |t|
  t.files   = ['features/**/*', 'lib/**/*.rb']
  t.options = ['--private']
end