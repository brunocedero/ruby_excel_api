# ruby_excel_api
Script en Ruby para implementación en APIRest

###### xBruno

El siguiente script está hecho en Ruby On Rails, su implememtación es leer de un archivo excel una lista de ID que podrían ser por ejemplo emnpleados o productos, luego utilizando la gema Httparty podemos hacer uso de métodos HTTP con los datos del archivo excel.

## Haremos uso de las siguientes gemas:
```
  gem 'roo', '~> 2.8.0'
  gem 'httparty'
```

- #### La gema roo es la utilizamos para leer archivos excel con Ruby, aquí su documentación:
 https://rubygems.org/gems/roo/
 
```ruby
#!/usr/bin/env ruby
# Created by: Bruno Cedero
# Read file and put the end date to employees from the list 

require 'bundler/inline'
require 'roo'
require 'httparty'
require 'uri'
require 'date'
require 'mongo'

gemfile do
  source 'https://rubygems.org'
  gem 'roo', '~> 2.8.0'
  gem 'httparty'
  gem 'byebug'
end

#file passed by console stored in ARGV(array),
#in the first position
file_path = ARGV[0]
employees_file = Roo::Spreadsheet.open(file_path)

# Create a hash, to acces columns by name
headers={}
employees_file.row(1).each_with_index {|header,i| headers[header] = i}

rows = {}
((employees_file.first_row + 1)..employees_file.last_row).each_with_index do |row, i|
  @employee_id = employees_file.row(row)[headers['employee_id']]
  @end_date = DateTime.now()
  data = {}
  data.store('employee_id', @employee_id)
  data.store('end_date', @end_date)
  rows[i] = data
end

token = 'token'
base_uri = 'http://localhost:3001'

#added end date by id
rows.each do |employee_to_update|
  employee_id = employee_to_update[1]['employee_id']
  employee_end_date = employee_to_update[1]['end_date']

  employee_data = {}
  employee_data['employee_id'] = employee_id
  employee_data['end_date'] = employee_end_date
  endpoint = URI("#{base_uri}/api/v1/employee/#{employee_data['employee_id']}")
  response = HTTParty.put(endpoint, body: employee_data.to_json, headers: { 'Content-Type' => 'application/json',
    'Authorization' => 'Bearer ' + token })
  
    if response.code != 200
      puts "ERROR --> #{response['error']} Operation aborted."
    else
      puts "Added end date"
    end
end

```
