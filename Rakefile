# *********************************************************************************
# URBANopt™, Copyright (c) 2019-2022, Alliance for Sustainable Energy, LLC, and other
# contributors. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without modification,
# are permitted provided that the following conditions are met:
#
# Redistributions of source code must retain the above copyright notice, this list
# of conditions and the following disclaimer.
#
# Redistributions in binary form must reproduce the above copyright notice, this
# list of conditions and the following disclaimer in the documentation and/or other
# materials provided with the distribution.
#
# Neither the name of the copyright holder nor the names of its contributors may be
# used to endorse or promote products derived from this software without specific
# prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
# ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
# WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED.
# IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT,
# INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING,
# BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
# LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE
# OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED
# OF THE POSSIBILITY OF SUCH DAMAGE.
# *********************************************************************************

require 'openstudio/extension'
require 'openstudio/extension/rake_task'
require 'urbanopt/scenario'
require 'urbanopt/geojson'
require 'urbanopt/reopt'
require 'urbanopt/reopt_scenario'

module URBANopt
  module ExampleGeoJSONProject
    class ExampleGeoJSONProject < OpenStudio::Extension::Extension

    end
  end
end

def configure_project(json)
  # write a runner.conf in project dir if it does not exist
  # delete runner.conf to automatically regenerate it
  project_name = File.basename(json, '.json')
  root_dir = File.join(Dir.pwd, 'projects', project_name)
  options = { gemfile_path: File.join(root_dir, 'Gemfile'), bundle_install_path: File.join(root_dir, '.bundle/install'), num_parallel: 7 }
  runner_file_path = File.join(root_dir, 'runner.conf')
  # write a runner.conf in project dir (if it does not already exist)
  if !File.exist?(runner_file_path)
    puts 'GENERATING runner.conf file'
    OpenStudio::Extension::RunnerConfig.init(root_dir) # itinialize the file with default values
    run_config = OpenStudio::Extension::RunnerConfig.new(root_dir) # get the configs
    # update paths
    options.each do |key, val|
      run_config.update_config(key, val) # update gemfile_path
    end
    # save back to disk
    run_config.save
  else
    runner_conf_hash = JSON.parse(File.read(runner_file_path))
    puts 'USING existing runner.conf file'
    if runner_conf_hash[:gemfile_path].empty?
      runner_conf_hash[:gemfile_path] = options[:gemfile_path]
    end
    if runner_conf_hash[:bundle_install_path].empty?
      runner_conf_hash[:gemfile_path] = options[:bundle_install_path]
    end

  end
end

def create_project(json)
  folder_name = json.split(".")[0]
  project_folder = File.join(Dir.pwd, "projects")
  dir_name = File.join(project_folder,folder_name)
  unless Dir.exists?(project_folder)
    FileUtils.mkdir project_folder
  end
  unless Dir.exists?(dir_name)
    FileUtils.mkdir File.join('projects', folder_name)
    FileUtils.mkdir File.join('projects', folder_name, 'weather')
    FileUtils.mkdir File.join('projects', folder_name, 'mappers')

    # copy config file
    FileUtils.cp(File.join('example_files', 'runner.conf'), File.join(dir_name, 'runner.conf'))

    # copy gemfile
    FileUtils.cp(File.join('example_files', 'Gemfile'), dir_name)

    # copy weather files
    weather_files = File.join('example_files', 'weather')
    Pathname.new(weather_files).children.each { |weather_file| FileUtils.cp(weather_file, File.join(dir_name, 'weather')) }

    # copy feature file
    FileUtils.cp(File.join('example_files', 'feature_files', json), dir_name)

    # copy residential files
    FileUtils.cp_r(File.join('example_files', 'residential'), File.join(dir_name, 'mappers', 'residential'))
    FileUtils.cp_r(File.join('example_files', 'measures'), File.join(dir_name, 'measures'))
    FileUtils.cp_r(File.join('example_files', 'resources'), File.join(dir_name, 'resources'))
    FileUtils.cp_r(File.join('example_files', 'xml_building'), File.join(dir_name, 'xml_building'))

  else
    puts "Project folder already exists..."
  end
end

def create_scenario_file(json, mapper, osw)
  folder_name = json.split(".")[0]
  dir_name = File.join(Dir.pwd, "projects/#{folder_name}")

  feature_file_json = JSON.parse(File.read(File.expand_path(File.join(dir_name,json))), symbolize_names: true)

  mapper_name = mapper.split(".")[0]
  scenario_file_name = "#{mapper_name.downcase}_scenario.csv"
  # Create CSV
  CSV.open(File.join(dir_name, scenario_file_name), 'wb', write_headers: true, headers: ['Feature Id', 'Feature Name', 'Mapper Class']) do |csv|
    begin
      feature_file_json[:features].each do |feature|
        # ensure that feature is a building
        if feature[:properties][:type] == 'Building'
          csv << [feature[:properties][:id], feature[:properties][:name], "URBANopt::Scenario::#{mapper_name}Mapper"]
        end
      end
    rescue NoMethodError
    
    abort("\nOops! You didn't provde a valid feature_file. Please provide path to the geojson feature_file")
    end
  
  end

  # copy mapper
  FileUtils.cp(File.join('example_files', 'mappers', mapper), File.join(dir_name, 'mappers'))

  # copy osw
  FileUtils.cp(File.join('example_files', 'osw', osw), File.join(dir_name, 'mappers'))

end


def run_project(feature_file, csv_file)
  folder_name = feature_file.split(".")[0]
  root_dir = File.join(Dir.pwd, "projects/#{folder_name}")
  mapper_name = csv_file.split(".")[0]
  run_dir = File.join(root_dir, "run/#{mapper_name}/")
  mapper_files_dir = File.join(root_dir, 'mappers')
  name = csv_file.split(".")[0]
  csv_file_dir = File.join(root_dir, csv_file)
  num_header_rows = 1
  feature_file_path = File.join(root_dir, feature_file)
  feature_file = URBANopt::GeoJSON::GeoFile.from_file(feature_file_path)
  scenario = URBANopt::Scenario::ScenarioCSV.new(name, root_dir, run_dir, feature_file, mapper_files_dir, csv_file_dir, num_header_rows)
  return scenario

end


# Load in the rake tasks from the base extension gem
#rake_task = OpenStudio::Extension::RakeTask.new
#rake_task.set_extension_class(URBANopt::ExampleGeoJSONProject::ExampleGeoJSONProject)

desc 'Create project'
task :urbanopt_create_project, [:json] do |t, args|
  puts 'Creating project...'

  json = args[:json]
  json = 'prototype_district_A.json' if json.nil?

  create_project(json)
end

desc 'Create scenario'
task :urbanopt_create_scenario, [:json, :mapper, :osw] do |t, args|
  puts 'Creating scenario...'

  json = args[:json]
  json = 'prototype_district_A.json' if json.nil?
  mapper = args[:mapper]
  mapper = 'Baseline.rb' if mapper.nil?
  osw = args[:osw]
  osw = 'base_workflow.odw' if osw.nil?

  create_scenario_file(json, mapper, osw)

end

desc 'Run project'
task :urbanopt_run_project, [:json, :csv] do |t, args|
  puts 'Running project...'

  feature_file = args[:json]
  csv_file = args[:csv]
  feature_file = 'prototype_district_A.json' if feature_file.nil?
  csv_file = 'baseline_scenario.csv' if csv_file.nil?

  #configure_project(feature_file)

  puts "\nSimulating features of '#{feature_file}' as directed by '#{csv_file}'...\n\n"
  scenario_runner = URBANopt::Scenario::ScenarioRunnerOSW.new
  scenario_runner.run(run_project(feature_file, csv_file))
end

desc 'Quick Test'
task :qt, [:json, :csv] do |t, args|
  puts 'Quick Test, creating project, creating scenario, running project, and post processing ...'

  Rake::Task["urbanopt_create_project"].invoke
  Rake::Task["urbanopt_create_scenario"].invoke
  Rake::Task["urbanopt_run_project"].invoke
  Rake::Task["urbanopt_post_process"].invoke

end

# todo - setup a weather sweep rake task like qt, but have scenario setup that changes weather file for each scenario
# make sure that change building location has climate zone to automatically set based on weather file
# change to smaller geojson file with two office buildings so quick run and
desc 'Climate Sweep'
task :clsw, [:json] do |t, args|
  puts 'Climate Sweep, Similar to Quick Test but this meakes a Scenario for each weather file in directory'

  # set default if JSON not passed in
  json = args[:json]
  json = 'prototype_district_A.json' if json.nil? #'proto_dist_a_min.json'

  # make one project for all scenarios
  Rake::Task["urbanopt_create_project"].invoke(json)

  # find and loop through all EPW files in weather directory
  weather_files = Dir["example_files/weather/*.epw"]
  puts "Found #{weather_files.size} in the example files"
  weather_files.each do |epw|

    basename = File.basename(epw)
    sweep_prefix = basename.split(".").first.gsub("USA_","").split("-").first
    mapper = "SweepBaseline_#{sweep_prefix}.rb"
    osw = "sweep_base_workflow.osw"

    puts "Creating and running Scenario for #{sweep_prefix}"
    Rake::Task["urbanopt_create_scenario"].invoke(json, mapper, osw)
    Rake::Task["urbanopt_create_scenario"].reenable # this lets invoke run again, can try execute instead which doesn't need this but that isn't working

    # name of CSV that should be made with scenario
    csv = "sweepbaseline_#{sweep_prefix.downcase}_scenario.csv"
    Rake::Task["urbanopt_run_project"].invoke(json, csv)
    Rake::Task["urbanopt_run_project"].reenable

    # currently if post processing fails the rake task ends, maybe find way to rescue this so the rest of the scenarios keep running
    # int falls goes fine through this but Atlanta fail with stange path with it looking for nested projects directories that do not exist.
    #Rake::Task["urbanopt_post_process"].invoke(json, csv)
    #Rake::Task["urbanopt_post_process"].reenable

  end

end

desc 'Post Process Project'
task :urbanopt_post_process, [:json, :csv] do |t, args|
  puts 'Post Processing Project...'

  json = args[:json]
  csv = args[:csv]
  json = 'prototype_district_A.json' if json.nil?
  csv = 'baseline_scenario.csv' if csv.nil?

  default_post_processor = URBANopt::Scenario::ScenarioDefaultPostProcessor.new(run_project(json, csv))
  scenario_result = default_post_processor.run
  # save scenario reports
  scenario_result.save
  # save feature reports
  scenario_result.feature_reports.each(&:save_json_report)
  scenario_result.feature_reports.each(&:save_csv_report)
end


task :default => :update_all
