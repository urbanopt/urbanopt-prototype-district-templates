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
require 'json' 

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

def visualize_features(feature_file, scenario_file)
  name = 'Visualize Feature Results'

  feature_file = File.basename(feature_file, File.extname(feature_file))
  scenario_name = File.basename(scenario_file, File.extname(scenario_file))
  root_dir = File.join(Dir.pwd, 'projects', feature_file)
  run_dir = File.join(root_dir, 'run', scenario_name.downcase)
  feature_report_exists = false
  csv = CSV.read(File.join(root_dir, scenario_file), headers: true)
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
    else puts "\nERROR: Default reports not created for #{feature}. Please use 'process --default' to create default post processing reports for all features first. Visualization not generated for #{feature}.\n"
    end
  end
  if feature_report_exists == true
    puts "\nCreating visualizations for Feature results in the Scenario\n"
    URBANopt::Scenario::ResultVisualization.create_visualization(feature_folders, true, feature_names)
    html_in_path = File.join(root_dir, 'visualization', 'input_visualization_feature.html')
    html_out_path = File.join(root_dir, 'run', scenario_name, 'feature_comparison.html')
    FileUtils.cp(html_in_path, html_out_path)
    puts "\nDone\n"
  end
end

# Load in the rake tasks from the base extension gem
#rake_task = OpenStudio::Extension::RakeTask.new
#rake_task.set_extension_class(URBANopt::ExampleGeoJSONProject::ExampleGeoJSONProject)

desc 'Create project'
task :urbanopt_create_project, [:json] do |t, args|
  puts 'Creating project...'

  json = args[:json]
  json = 'urban_edge_example.json' if json.nil?

  create_project(json)
end

desc 'Create scenario'
task :urbanopt_create_scenario, [:json, :mapper, :osw] do |t, args|
  puts 'Creating scenario...'

  json = args[:json]
  json = 'urban_edge_example.json' if json.nil?
  mapper = args[:mapper]
  mapper = 'BaselineModified.rb' if mapper.nil?
  osw = args[:osw]
  osw = 'base_workflow.osw' if osw.nil?

  create_scenario_file(json, mapper, osw)

end

desc 'Run project'
task :urbanopt_run_project, [:json, :csv] do |t, args|
  puts 'Running project...'

  feature_file = args[:json]
  csv_file = args[:csv]
  feature_file = 'urban_edge_example.json' if feature_file.nil?
  csv_file = 'baselinemodified_scenario.csv' if csv_file.nil?

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
  json_raw = args[:json]
  json_raw = 'urban_edge_example.json' if json_raw.nil?

  # create a copy of JSON file with temp name
  project_folder = File.join(Dir.pwd, "example_files/feature_files")

  # path to go back to later
  orig_dir = Dir.pwd

  # find and loop through all EPW files in weather directory
  weather_files = Dir["example_files/weather/*.epw"]
  puts "Found #{weather_files.size} in the example files"
  weather_files.sort.each do |epw|

    basename = File.basename(epw)
    sweep_prefix = basename.split(".").first.gsub("USA_","").split("-").first
    mapper = "SweepBaselineModified.rb"
    osw = "sweep_base_workflow.osw"

    # alter copy of JSON file to update weather file name
    FileUtils.cp(File.join(project_folder, json_raw), File.join(project_folder, "sweep_#{sweep_prefix.downcase}.json"))
    json_mod_name = "sweep_#{sweep_prefix.downcase}.json"
    json_mod = JSON.parse(File.read(File.join(project_folder, json_mod_name,)))
    json_mod["project"]["weather_filename"] = "#{basename}"
    json_mod["project"]["climate_zone"] = nil

    json_mod["features"].first["properties"]["weather_filename"] = "#{basename}"
    json_mod["features"].first["properties"]["climate_zone"] = nil

    # saving altered GeoJSON
    json_mod.to_json
    json_mod_path = File.join(project_folder, json_mod_name)
    puts "Updating temp GEOJSON file for  #{json_mod_name}"
    File.open(json_mod_path, "w") do |f|
      f.puts JSON.pretty_generate(json_mod)
    end

    # make one project for each weather file
    puts "Creating project for #{sweep_prefix}"
    Rake::Task["urbanopt_create_project"].invoke(json_mod_name)
    Rake::Task["urbanopt_create_project"].reenable # this lets invoke run again, can try execute instead which doesn't need this but that isn't working

    # cleanup file file in example_files/feature_files
    File.delete(json_mod_path)

    puts "Creating and running Scenario for #{sweep_prefix}"
    Rake::Task["urbanopt_create_scenario"].invoke(json_mod_name, mapper, osw)
    Rake::Task["urbanopt_create_scenario"].reenable # this lets invoke run again, can try execute instead which doesn't need this but that isn't working

    # name of CSV that should be made with scenario
    csv = "sweepbaselinemodified_scenario.csv"
    Rake::Task["urbanopt_run_project"].invoke(json_mod_name, csv)
    Rake::Task["urbanopt_run_project"].reenable

    # post process fails to find GeoJSON file turned off for now
    #GeoJSON file '/Users/dgoldwas/Documents/github/uo/proto_dist/projects/sweep_mn_international/projects/sweep_mn_international/sweep_mn_international.json' does not exist

    # currently if post processing fails the rake task ends, maybe find way to rescue this so the rest of the scenarios keep running
    # int falls goes fine through this but Atlanta fail with stange path with it looking for nested projects directories that do not exist.
    #Rake::Task["urbanopt_post_process"].invoke(json_mod_name, csv)
    #Rake::Task["urbanopt_post_process"].reenable

    # Seems like need to move back up 2 directories because next location gets made within this project instead of at top level
    Dir.chdir(orig_dir)

  end

end

desc 'Post Process Project'
task :urbanopt_post_process, [:json, :csv] do |t, args|
  puts 'Post Processing Project...'

  json = args[:json]
  csv = args[:csv]
  json = 'urban_edge_example.json' if json.nil?
  csv = 'baselinemodified_scenario.csv' if csv.nil?

  default_post_processor = URBANopt::Scenario::ScenarioDefaultPostProcessor.new(run_project(json, csv))
  scenario_result = default_post_processor.run
  # save scenario reports
  scenario_result.save
  # save feature reports
  scenario_result.feature_reports.each(&:save_json_report)
  scenario_result.feature_reports.each(&:save_csv_report)
end

# Visualize feature results

desc 'Visualize and compare results for all Features in a Scenario'
task :urbanopt_visualize_features, [:json, :csv] do |t, args|
  puts 'Visualizing results for all Features in the Scenario...'

  json = args[:json]
  csv = args[:csv]
  json = 'urban_edge_example.json' if json.nil?
  csv = 'baselinemodified_scenario.csv' if csv.nil?

  visualize_features(json, csv)
end

task :default => :update_all

# Create Testing Summary Report for climate sweep
# This expects all projects to be run from same GeoJSON with same featureID's
# First column is ID, followed by EUI for each climate zone, then unmet horus for each climate zone, then feature runtime min/max, E+ runtime min/max

desc 'Generuate Climate Zone Sweep Summary CSV'
task :clsw_summary do
  puts 'Making CSV across all run projects with feature ID as key; first column'

  # test specific strigns used for climate zone sweep
  project_prefix = "sweep_"
  scenario = "sweepbaselinemodified_scenario"

  # hash that will be converted to CSV
  feature_hash = {}
  cz_strings = []

  # loop through projects
  Dir["projects/#{project_prefix}*"].sort.each do |project|
    if Dir.exist?("#{project}/run/#{scenario}")
      cz_string = project.gsub("projects/#{project_prefix}","")
      puts "* Gathering data for #{cz_string}"
      Dir["#{project}/run/#{scenario}/*"].sort.each do |feature|
        next if ! File.directory?(feature)
        feature_id = File.basename(feature)
        # create new entry if this feature_id hasn't been added yet
        if !feature_hash.has_key?(feature_id)
          feature_hash[feature_id] = {:eui => {}, :unmet_occ => {}, :feature_rt => {}, :ep_rt => {}}
        end

        # pouplate data for this feature for this location
        feature_hash[feature_id][:eui][cz_string] = 0.0 # todo - add lookup later
        feature_hash[feature_id][:unmet_occ][cz_string] = 0.0 # todo - add lookup later
        feature_hash[feature_id][:feature_rt][cz_string] = 0.0 # todo - add lookup later
        feature_hash[feature_id][:ep_rt][cz_string] = 0.0 # todo - add lookup later

      end
    end
  end

  # for diagnostics, structured differentlY (nested) than csv
  # puts feature_hash.inspect

  # populate csv
  require "csv"
  csv_rows = []
  made_headers = false
  headers = ['feature_id']
  feature_hash.each do |feature_id,v|
    arr_row = [feature_id]

    v[:eui].each do |cz,v_eui|
      arr_row << v_eui
      if ! made_headers then headers << "eui_#{cz}" end
    end
    v[:unmet_occ].each do |cz,v_unmet_occ|
      arr_row << v_unmet_occ
      if ! made_headers then headers << "unmet_occ_#{cz}" end
    end
    v[:feature_rt].each do |cz,v_feature_rt|
      arr_row << v_feature_rt
      if ! made_headers then headers << "feature_rt_#{cz}" end
    end
    v[:ep_rt].each do |cz,v_ep_rt|
      arr_row << v_ep_rt
      if ! made_headers then headers << "ep_rt_#{cz}" end
    end
    made_headers = true

    csv_row = CSV::Row.new(headers, arr_row)
    csv_rows.push(csv_row)  
  end

  # save csv
  csv_table = CSV::Table.new(csv_rows)
  path_report = "projects/climate_sweep_overview.csv"
  puts "saving csv file to #{path_report}"
  File.open(path_report, 'w'){|file| file << csv_table.to_s}

end