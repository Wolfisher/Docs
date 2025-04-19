# Useful notes to learn SPL
## About `eval` command
### Basic Syntax:
`... | eval <new_field>=<expression>`

### Common use cases
1. Create derived fields:  
` ... | eval total_price=price*quantity`
2. String operations:  
`... | eval username=lower(user)`
3. Conditional logic (if-else):  
`... | eval status=if(code==200, "OK", "Error")`
4. Convert data types:  
`... | eval timestamp=strptime(_time, "%Y-%m-%d %H:%M:%S")`
5. Date math:  
`... | eval days_since_event=(now() - _time)/86400`

### Dashboard examples  
```sql
| inputlookup some-inventory.csv 
| where filter1=$filter1$ 
| join type=left keyword1 
    [ search index=index1 OR index=index2 keyword2 
    | dedup keyword1 
    | convert ctime(_time) as startupTime] 
| join type=left keyword2
    [ search index=index1 OR index=index2 logger=""  port  | table field1,field2,field3,field4
| where ("$module$"="all" OR module="$module$")
| table field1,field2,field3,field4
```