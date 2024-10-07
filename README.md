# solr-apache
### Function Definitions
createCollection(p_collection_name):
Creates a new Solr collection with the specified name.

indexData(p_collection_name, p_exclude_column):
Indexes employee data into the specified collection, excluding the given column.

searchByColumn(p_collection_name, p_column_name, p_column_value):
Searches for records in the specified collection where the given column matches the provided value.

getEmpCount(p_collection_name):
Retrieves the total number of employees in the specified collection.

delEmpById(p_collection_name, p_employee_id):
Deletes the employee with the specified ID from the collection.

getDepFacet(p_collection_name):
Retrieves the count of employees grouped by department from the specified collection
### program
```
import json
import requests
import pandas as pd
```
### Function to create a new collection
```
def create_collection(collection_name):
    url = f"http://localhost:8983/solr/admin/collections?action=CREATE&name={collection_name}&numShards=1&replicationFactor=1"
    response = requests.get(url).json()
    print(f"Create Collection Response: {response}")
```
### Adding a schema for 'Department'
```
    schema_url = f"http://localhost:8983/solr/{collection_name}/schema"
    headers = {"Content-Type": "application/json"}
    schema_data = {
        "add-field": {
            "name": "Department",
            "type": "string",
            "multiValued": False
        }
    }
    schema_response = requests.post(schema_url, headers=headers, data=json.dumps(schema_data))
    print(f"Schema Addition Response: {schema_response.status_code}")
```
### Function to index data into the collection, excluding a specific column
```
def index_data(collection_name, exclude_column):
    df = pd.read_csv("Employee.csv", encoding='latin-1')
    df = df.drop(columns=[exclude_column])
    df.columns = df.columns.str.replace(' ', '_')
    csv_data = df.to_csv(index=False, encoding='utf-8')

    url = f"http://localhost:8983/solr/{collection_name}/update?commit=true"
    headers = {"Content-Type": "application/csv"}
    response = requests.post(url, headers=headers, data=csv_data)
    
    print(f"Data Indexed: \n{df}")
    print(f"Response Status Code: {response.status_code}")
    print(f"Response Content: {response.text}")
```
### Function to search the collection by a specific column and value
```
def search_by_column(collection_name, column_name, column_value):
    url = f"http://localhost:8983/solr/{collection_name}/select?indent=true&q={column_name}:{column_value}&wt=json"
    response = requests.get(url)
    results = response.json()['response']['docs']
    formatted_json = json.dumps(results, indent=4)
    print(f"Search Results: \n{formatted_json}")
```
# Function to get the count of employees in the collection
```
def get_employee_count(collection_name):
    url = f"http://localhost:8983/solr/{collection_name}/select?q=*:*&rows=0"
    response = requests.get(url)
    employee_count = response.json()['response']['numFound']
    print(f"Total Employees in {collection_name}: {employee_count}")
```
### Function to delete an employee by ID
```
def delete_employee_by_id(collection_name, employee_id):
    url = f"http://localhost:8983/solr/{collection_name}/update?commit=true"
    data = json.dumps({'delete': {'query': f"Employee_ID:{employee_id}"}})
    headers = {"Content-Type": "application/json"}
    response = requests.post(url, headers=headers, data=data)
    
    print(f"Delete Response Status Code: {response.status_code}")
    print(f"Delete Response Content: {response.text}")
```
### Function to get department facets
```
def get_department_facets(collection_name):
    url = f"http://localhost:8983/solr/{collection_name}/select?q=*:*&facet=true&facet.field=Department"
    response = requests.get(url)
    facets = response.json()['facet_counts']['facet_fields']['Department']
    print(f"Department Facets in {collection_name}: {facets}")

# Execute the functions with the given parameters
v_name_collection = "pragatheevran"
v_phone_collection = "0788"
```
```
create_collection(v_name_collection)
create_collection(v_phone_collection)
get_employee_count(v_name_collection)
index_data(v_name_collection, 'Department')
index_data(v_phone_collection, 'Gender')
delete_employee_by_id(v_name_collection, 'E02003')
get_employee_count(v_name_collection)
search_by_column(v_name_collection, 'Department', 'IT')
search_by_column(v_name_collection, 'Gender', 'Male')
search_by_column(v_phone_collection, 'Department', 'IT')
get_department_facets(v_name_collection)
get_department_facets(v_phone_collection)

```
### output:
Collection Creation:
Sends a request to create a Solr collection and defines a schema for the "Department" field.

![image](https://github.com/user-attachments/assets/0bd9fc9a-2acb-4126-b4bc-72be155f83c7)

Index Data:
Reads the employee data from a CSV file, drops the excluded column, and indexes the remaining data into Solr.
![Screenshot 2024-10-08 000146](https://github.com/user-attachments/assets/bd45c11d-4d78-4c44-9da4-dbb4a8b5a3a4)


![image](https://github.com/user-attachments/assets/cb402dfc-1e1d-4977-a9cb-9aae1e375740)

Search By Column:
Queries Solr for records where the specified column matches the given value and formats the results.
![Screenshot 2024-10-08 011737](https://github.com/user-attachments/assets/1c5ef8e0-cd2c-4e50-b9e2-6b6c67c728d3)
![Screenshot 2024-10-08 011547](https://github.com/user-attachments/assets/fd9ed3ed-f23c-439c-83c6-bdadb0f32221)


Employee Count:
Retrieves the total number of employees in the collectio![Screenshot 2024-10-08 003754](https://github.com/user-attachments/assets/b6c3e308-8a23-41a3-bb71-62d636aa3b4d)
employee id:
![Screenshot 2024-10-08 010525](https://github.com/user-attachments/assets/97657622-8c94-47e9-a662-1f7ec98e648a)

n by querying Solr.

Delete By Employee ID:
Sends a request to delete a specific employee by their ID.
![Screenshot 2024-10-08 010525](https://github.com/user-attachments/assets/47f3173b-0dc2-42aa-b65c-05dda33975c3)


Department Facets:
Retrieves the count of employees grouped by department.
![Screenshot 2024-10-08 012456](https://github.com/user-attachments/assets/6894cc96-6ab1-4f01-a948-f8b446e2cabb)

