require 'bundler'
require 'rake/clean'
require 'rake/testtask'
gem 'rdoc'
require 'rdoc/task'
require 'roodi'
require 'roodi_task'
require 'cucumber'
require 'cucumber/rake/task'

old_verbose = $VERBOSE
$VERBOSE=0
require 'reek/rake/task'
Reek::Rake::Task.new do |t|
  t.fail_on_error = false
  t.config_files = ['test/gli.reek']
  t.ruby_opts=['-W0']
  t.reek_opts=['-q']
  t.source_files = FileList['lib/**/*.rb'] - ['lib/gli/commands/scaffold.rb','lib/gli/commands/help.rb'] - FileList['lib/gli/commands/help_modules/*.rb']
end
$VERBOSE=old_verbose

include Rake::DSL

CLEAN << "log"
CLOBBER << FileList['**/*.rbc']

Rake::RDocTask.new do |rd|
  rd.main = "README.rdoc"
  rd.rdoc_files = FileList["lib/**/*.rb","README.rdoc"] - 
                  FileList["lib/gli/commands/help_modules/*.rb"] - 
                  ["lib/gli/commands/help.rb",
                   "lib/gli/commands/scaffold.rb",
                   "lib/gli/support/*.rb",
                   "lib/gli/app_support.rb",
                   "lib/gli/option_parser_factory.rb",
                   "lib/gli/gli_option_parser.rb",
                   "lib/gli/command_support.rb",]
  rd.title = 'GLI - Git Like Interface for your command-line apps'
end

Bundler::GemHelper.install_tasks

RoodiTask.new do |t|
  t.patterns = ['lib/**/*.rb']
  t.config = 'test/roodi.yaml'
end

desc 'run unit tests'
Rake::TestTask.new do |t|
  t.libs << "test"
  t.test_files = FileList['test/init_simplecov.rb','test/tc_*.rb']
end

CUKE_RESULTS = 'results.html'
CLEAN << CUKE_RESULTS
Cucumber::Rake::Task.new(:features) do |t|
  opts = "features --format html -o #{CUKE_RESULTS} --format progress -x"
  opts += " --tags #{ENV['TAGS']}" if ENV['TAGS']
  t.cucumber_opts =  opts
  t.fork = false
end
Cucumber::Rake::Task.new('features:wip') do |t|
  tag_opts = ' --tags ~@pending'
  tag_opts = ' --tags @wip'
  t.cucumber_opts = "features --format html -o #{CUKE_RESULTS} --format pretty -x -s#{tag_opts}"
  t.fork = false
end

begin
  require 'rcov/rcovtask'
  task :clobber_coverage do
    rm_rf "coverage"
  end

  desc 'Measures test coverage'
  task :coverage => :rcov do
    puts "coverage/index.html contains what you need"
  end

  Rcov::RcovTask.new do |t|
    t.libs << 'lib'
    t.test_files = FileList['test/tc_*.rb']
  end
rescue LoadError
  begin
    require 'simplecov'
  rescue LoadError
    $stderr.puts "neither rcov nor simplecov are installed; you won't be able to check code coverage"
  end
end

desc 'Publish rdoc on github pages and push to github'
task :publish_rdoc => [:rdoc,:publish]

task :default => [:test,:features,:roodi]

