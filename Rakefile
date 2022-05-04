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

      # number of datapoints(features) you want to run in parallel
      # based on the number of available processors on your local machine.
      # This does not seem to function, instead added line to code for runner.config file
      # OpenStudio::Extension::Extension::NUM_PARALLEL = 7

      # set MAX_DATAPOINTS
      OpenStudio::Extension::Extension::MAX_DATAPOINTS = 1000

      def initialize
        super
        @root_dir = File.absolute_path(File.join(File.dirname(__FILE__), 'example_files'))
      end

      # Return the absolute path of the measures or empty string if there is none, can be used when configuring OSWs
      def measures_dir
        ""
      end

      # Relevant files such as weather data, design days, etc.
      # Return the absolute path of the files or nil if there is none, used when configuring OSWs
      def files_dir
        return File.absolute_path(File.join(@root_dir, 'weather'))
      end

    end
  end
end

def root_dir
  return File.join(File.dirname(__FILE__), 'example_project')
end

def baseline_scenario(json, csv)
  name = 'Baseline Scenario'
  scenario = File.basename(csv, '.csv')
  run_dir = File.join(root_dir, "run/#{scenario}/")
  feature_file_path = File.join(root_dir, json)
  csv_file = File.join(root_dir, csv)
  mapper_files_dir = File.join(root_dir, 'mappers/')
  num_header_rows = 1
  feature_file = URBANopt::GeoJSON::GeoFile.from_file(feature_file_path)
  scenario = URBANopt::Scenario::ScenarioCSV.new(name, root_dir, run_dir, feature_file, mapper_files_dir, csv_file, num_header_rows)
  return scenario
end

def high_efficiency_scenario(json, csv)
  name = 'High Efficiency Scenario'
  run_dir = File.join(root_dir, 'run/high_efficiency_scenario/')
  feature_file_path = File.join(root_dir, json)
  csv_file = File.join(root_dir, csv)
  mapper_files_dir = File.join(root_dir, 'mappers/')
  num_header_rows = 1
  feature_file = URBANopt::GeoJSON::GeoFile.from_file(feature_file_path)
  scenario = URBANopt::Scenario::ScenarioCSV.new(name, root_dir, run_dir, feature_file, mapper_files_dir, csv_file, num_header_rows)
  return scenario
end

def thermal_storage_scenario(json, csv)
  name = 'Thermal Storage Scenario'
  run_dir = File.join(root_dir, 'run/thermal_storage_scenario/')
  feature_file_path = File.join(root_dir, json)
  csv_file = File.join(root_dir, csv)
  mapper_files_dir = File.join(root_dir, 'mappers/')
  num_header_rows = 1
  feature_file = URBANopt::GeoJSON::GeoFile.from_file(feature_file_path)
  scenario = URBANopt::Scenario::ScenarioCSV.new(name, root_dir, run_dir, feature_file, mapper_files_dir, csv_file, num_header_rows)
  return scenario
end

def flexible_hot_water_scenario(json, csv)
  name = 'Flexible Hot Water Scenario'
  run_dir = File.join(root_dir, 'run/flexiblehotwater_scenario/')
  feature_file_path = File.join(root_dir, json)
  csv_file = File.join(root_dir, csv)
  mapper_files_dir = File.join(root_dir, 'mappers/')
  num_header_rows = 1
  feature_file = URBANopt::GeoJSON::GeoFile.from_file(feature_file_path)
  scenario = URBANopt::Scenario::ScenarioCSV.new(name, root_dir, run_dir, feature_file, mapper_files_dir, csv_file, num_header_rows)
  return scenario
end

def mixed_scenario(json, csv)
  name = 'Mixed Scenario'
  run_dir = File.join(root_dir, 'run/mixed_scenario/')
  feature_file_path = File.join(root_dir, json)
  csv_file = File.join(root_dir, csv)
  mapper_files_dir = File.join(root_dir, 'mappers/')
  num_header_rows = 1

  feature_file = URBANopt::GeoJSON::GeoFile.from_file(feature_file_path)
  scenario = URBANopt::Scenario::ScenarioCSV.new(name, root_dir, run_dir, feature_file, mapper_files_dir, csv_file, num_header_rows)
  return scenario
end

def reopt_scenario(json, csv)
  name = 'REopt Scenario'
  run_dir = File.join(root_dir, 'run/reopt_scenario/')
  feature_file_path = File.join(root_dir, json)
  csv_file = File.join(root_dir, csv)
  mapper_files_dir = File.join(root_dir, 'mappers/')
  reopt_files_dir = File.join(root_dir, 'reopt/')
  scenario_reopt_assumptions_file_name = 'base_assumptions.json'
  num_header_rows = 1

  feature_file = URBANopt::GeoJSON::GeoFile.from_file(feature_file_path)
  scenario = URBANopt::Scenario::REoptScenarioCSV.new(name, root_dir, run_dir, feature_file, mapper_files_dir, csv_file, num_header_rows, reopt_files_dir, scenario_reopt_assumptions_file_name)
  return scenario
end

def configure_project
  # write a runner.conf in project dir if it does not exist
  # delete runner.conf to automatically regenerate it
  options = {gemfile_path: File.join(root_dir, 'Gemfile'), bundle_install_path: File.join(root_dir, ".bundle/install"), num_parallel: 7}

  # write a runner.conf in project dir (if it does not already exist)
  if !File.exists?(File.join(root_dir, 'runner.conf'))
    puts "GENERATING runner.conf file"
    OpenStudio::Extension::RunnerConfig.init(root_dir)  # itinialize the file with default values
    run_config = OpenStudio::Extension::RunnerConfig.new(root_dir) # get the configs
    # update paths
    options.each do |key, val|
      run_config.update_config(key, val) # update gemfile_path
    end
    # save back to disk
    run_config.save
  else
    puts "USING existing runner.conf file"
  end
end

def visualize_scenarios
  name = 'Visualize Scenario Results'
  run_dir = File.join(root_dir, 'run')
  scenario_folders = []
  scenario_report_exists = false
  Dir.glob(File.join(run_dir, '/*_scenario')) do |scenario_folder|
    scenario_report = File.join(scenario_folder, 'scenario_optimization.csv')
    # Check if Scenario Optimization REopt file exists and add that
    if File.exist?(File.join(scenario_folder, 'scenario_optimization.csv'))
      scenario_folders << File.join(scenario_folder, 'scenario_optimization.csv')
      scenario_report_exists = true
    # Check if Default Feature Report exists and add that
    elsif File.exist?(File.join(scenario_folder, 'default_scenario_report.csv'))
      scenario_folders << File.join(scenario_folder, 'default_scenario_report.csv')
      scenario_report_exists = true
    elsif
      puts "\nERROR: Default reports not created for #{scenario_folder}. Please use 'process --default' to create default post processing reports for all scenarios first. Visualization not generated for #{scenario_folder}.\n"
    end
  end
  if scenario_report_exists == true
    puts "\nCreating visualizations for all Scenario results\n"
    URBANopt::Scenario::ResultVisualization.create_visualization(scenario_folders, false)
    vis_file_path = File.join(root_dir, 'visualization')
    if !File.exist?(vis_file_path)
      Dir.mkdir File.join(root_dir, 'visualization')
    end
    html_in_path = File.join(vis_file_path, 'input_visualization_scenario.html')
    if !File.exist?(html_in_path)
      $LOAD_PATH.each do |path_item|
        if path_item.to_s.end_with?('example_files')
          FileUtils.cp(File.join(path_item, 'visualization', 'input_visualization_scenario.html'), html_in_path)
        end
      end
    end
    html_out_path = File.join(run_dir, 'scenario_comparison.html')
    FileUtils.cp(html_in_path, html_out_path)
    puts "\nDone\n"
  end
end

def visualize_features(scenario_file)
  name = 'Visualize Feature Results'

  scenario_name = File.basename(scenario_file, File.extname(scenario_file))
  run_dir = File.join(root_dir, 'run', scenario_name.downcase)
  feature_report_exists = false
  csv = CSV.read(File.join(root_dir, scenario_file), :headers => true)
  feature_names = csv['Feature Name']
  feature_folders = []
  # loop through building feature ids from scenario csv
  csv['Feature Id'].each do |feature|
    # Check if Feature Optimization REopt file exists and add that
    if File.exist?(File.join(run_dir, feature, 'feature_reports/feature_optimization.csv'))
      feature_report_exists = true
      feature_folders << File.join(run_dir, feature, 'feature_reports/feature_optimization.csv')
    elsif File.exist?(File.join(run_dir, feature, 'feature_reports/default_feature_report.csv'))
      feature_report_exists = true
      feature_folders << File.join(run_dir, feature, 'feature_reports/default_feature_report.csv')
    elsif
      puts "\nERROR: Default reports not created for #{feature}. Please use 'process --default' to create default post processing reports for all features first. Visualization not generated for #{feature}.\n"
    end
  end
  if feature_report_exists == true
    puts "\nCreating visualizations for Feature results in the Scenario\n"
    URBANopt::Scenario::ResultVisualization.create_visualization(feature_folders, true, feature_names)
    vis_file_path = File.join(root_dir, 'visualization')
    if !File.exist?(vis_file_path)
      Dir.mkdir File.join(root_dir, 'visualization')
    end
    html_in_path = File.join(vis_file_path, 'input_visualization_feature.html')
    if !File.exist?(html_in_path)
      $LOAD_PATH.each do |path_item|
        if path_item.to_s.end_with?('example_files')
          FileUtils.cp(File.join(path_item, 'visualization', 'input_visualization_feature.html'), html_in_path)
        end
      end
    end
    html_out_path = File.join(root_dir, 'run', scenario_name, 'feature_comparison.html')
    FileUtils.cp(html_in_path, html_out_path)
    puts "\nDone\n"
  end
end

# Load in the rake tasks from the base extension gem
rake_task = OpenStudio::Extension::RakeTask.new
rake_task.set_extension_class(URBANopt::ExampleGeoJSONProject::ExampleGeoJSONProject)

desc 'Create project'
task :urbanopt_create_project, [:json] do |t, args|
  puts 'Creating project...'

  json = args[:json]
  json = 'prototype_district_A.json' if json.nil?

end

desc 'Create scenario csv'
task :urbanopt_create_scenario_csv, [:json] do |t, args|
  puts 'Creating scenario...'

  json = args[:json]
  json = 'prototype_district_A.json' if json.nil?

end

desc 'Run project'
task :urbanopt_run_project, [:json, :csv] do |t, args|
  puts 'Running project...'

  json = args[:json]
  csv = args[:csv]
  json = 'prototype_district_A.json' if json.nil?
  csv = 'baseline_scenario.csv' if csv.nil?

end

desc 'Post Process Project'
task :urbanopt_post_process, [:json, :csv] do |t, args|
  puts 'Post Processing Project...'

  json = args[:json]
  csv = args[:csv]
  json = 'example_project_combined.json' if json.nil?
  csv = 'baseline_scenario.csv' if csv.nil?

end


task :default => :update_all
