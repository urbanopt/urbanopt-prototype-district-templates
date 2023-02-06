# urbanopt-prototype-district-generator
URBANopt prototype district generator development in collaboration with LBNL and NREL

The URBANopt prototype district generator can be used to create and run URBANopt projects for prototype
district types. `urban_edge_example.json` contains building feature elments represenative of an urban edge district. It contains a variety of building types, sizing, number of stories, and vintages. For testing purpases with climate sweep task, 4 features are exclued from the scenario to expidte the time taken for testing.

The `BaselineModified.rb` and `SweepBaselineModified.rb` use a working that uses building type lookup to determine logic for geometry creation for commerical buildigns. In some cases a blending whole building space type is used with a core and perimeter zoning to GeoJSON feature footprint. In other cases discrete space types are used with a rectangular building that uses the GeoJSON feature footprint to get the building exposure by orientation.

The following commands can be utilized: 

**To view all commands**: 

```ruby
bundle exec rake -T
```

**To create a project**: 

```ruby
bundle exec rake urbanopt_create_project[<feature_file_name>.json]
```

**To create a scenario**:


```ruby
bundle exec rake urbanopt_create_scenario[<feature_file_name>.json,<mapper_name>.rb]
```


**To run a project**: 

```ruby
bundle exec rake urbanopt_run_project[<feature_file_name>.json,<scenario_name>.csv]
```

**To post process a project**:

```ruby
bundle exec rake urbanopt_post_process[<feature_file_name>.json,<scenario_name>.csv]
```

**To visualize features**:

```ruby
bundle exec rake urbanopt_visualize_features[<feature_file_name>.json,<scenario_name>.csv]
```

**To run entire workflow**: Currently hard coded to default GeoJSON file

```ruby
bundle exec rake urbanopt_full_workflow
```

**To run climate sweep across all CSV files**: If urban_edge_example.json is selected 4 feature buildings are disabled to speed up testing. Each cliamte zone will take 1-2 hours to run with the selected features disabled.

**To generate a csv summary file or climate sweep projects**: This does not require post-processing or visualization to be run. It generates a CSV file with a row for each feature_Id, and columns for building type, building area, min and max feature runtime, and min and max EnergyPlus runtime. Lastly itcontains EUI, Unmet Cooling Hours, and Unmet Heating hours for each climate zone run. As conifigured it results in 1168 building simulations.

```ruby
bundle exec rake urbanopt_climate_sweep_generate_summary_csv
```