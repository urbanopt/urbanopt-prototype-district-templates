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

def create_project(json)
  folder_name = json.split(".")[0]
  dir_name = File.join(Dir.pwd, "projects/#{folder_name}")
  unless Dir.exists?(dir_name)
    Dir.mkdir File.join('projects', folder_name)
    Dir.mkdir File.join('projects', folder_name, 'weather')
    Dir.mkdir File.join('projects', folder_name, 'mappers')

    # copy config file
    #FileUtils.cp(File.join('example_files', 'runner.conf'), File.join('projects', folder_name, 'runner.conf'))

    # copy gemfile
    FileUtils.cp(File.join('example_files', 'Gemfile'), dir_name)

    # copy weather files
    weather_files = File.join('example_files', 'weather')
    Pathname.new(weather_files).children.each { |weather_file| FileUtils.cp(weather_file, File.join(dir_name, 'weather')) }

    # copy feature file
    FileUtils.cp(File.join('example_files', 'feature_files', json), dir_name)

  end
end

def create_scenario_file(json, mapper)
  folder_name = json.split(".")[0]
  dir_name = File.join(Dir.pwd, "projects/#{folder_name}")

  feature_file_json = JSON.parse(File.read(File.expand_path(File.join(dir_name,json))), symbolize_names: true)

  mapper_name = mapper.split(".")[0]
  scenario_file_name = "#{mapper_name}_scenario.csv"
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
  FileUtils.cp(File.join('example_files', 'osw', 'base_workflow.osw'), File.join(dir_name, 'mappers'))

end


def run_project(feature_file, csv_file)
  folder_name = feature_file.split(".")[0]
  root_dir = File.join(Dir.pwd, "projects/#{folder_name}")
  mapper_name = csv_file.split(".")[0]
  run_dir = File.join(root_dir, "run/#{mapper_name}/")
  mapper_files_dir = File.join(root_dir, 'mappers')
  name = csv_file.split(".")[0]
  num_header_rows = 1
  feature_file_path = File.join(root_dir, feature_file)
  
  feature_file = URBANopt::GeoJSON::GeoFile.from_file(feature_file_path)
  scenario = URBANopt::Scenario::ScenarioCSV.new(name, root_dir, run_dir, feature_file, mapper_files_dir, csv_file, num_header_rows)
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
task :urbanopt_create_scenario, [:json, :mapper] do |t, args|
  puts 'Creating scenario...'

  json = args[:json]
  json = 'prototype_district_A.json' if json.nil?
  mapper = args[:mapper]
  mapper = 'Baseline.rb' if mapper.nil?

  create_scenario_file(json, mapper)

end

desc 'Run project'
task :urbanopt_run_project, [:json, :csv] do |t, args|
  puts 'Running project...'

  feature_file = args[:json]
  csv_file = args[:csv]
  feature_file = 'prototype_district_A.json' if feature_file.nil?
  csv_file = 'baseline_scenario.csv' if csv_file.nil?

  scenario_runner = URBANopt::Scenario::ScenarioRunnerOSW.new
  scenario_runner.run(run_project(feature_file, csv_file))

end

desc 'Quick Test'
task :qt, [:json, :csv] do |t, args|
  puts 'Quick Test, creating project, creating scenario, running project ...'

  Rake::Task["urbanopt_create_project"].invoke
  Rake::Task["urbanopt_create_scenario"].invoke
  Rake::Task["urbanopt_run_project"].invoke

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
