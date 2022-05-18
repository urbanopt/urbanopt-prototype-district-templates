# urbanopt-prototype-district-generator
URBANopt prototype district generator development in collaboration with LBNL and NREL

The URBANopt prototype district generator can be used to create and run URBANopt projects for prototype
district types.

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