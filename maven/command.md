Maven command to manage project versions:

1. Create a new version draft:  
    1. mvn versions:set -DnewVersion=<major>.<minor>.<patch>-SNAPSHOT
2. If test failed and you need to revert back:  
    1. mvn versions:revert
3. If test succeeded and you can proceed:  
    1. mvn versions:commit

