# SQLight


SQLight is a simple Swift layer over SQLite3.

Working with SQLite3 in Swift is usually a headache due to the data types used in C. 
SQLight handles all the headaches for you so that you can peacefully use it with Swift. 

### How is it different from other swift implementations?

hmmm...

Most of the famous GitHub projects on sqlite3-swift are too complex to begin for swift beginners.
And also it takes some time to learn the workings of the framework which **I think** it is *@#$@* waste of time.
..
..
..
many more

## Usage

#### Steps to follow:

1) Open database connection.
2) Prepare a statement using SQL query.
3) Do operations on prepared statement.
4) Destroy the prepared statement.
5) Close the database connection.

#### Code:

```swift
do
{
    var rowsChanged: Int
    
    /**************************************/
    // Open a connection to database
    let db = try SQLight(type: .file("test.db"))
    /**************************************/
    
    
    /**************************************/
    // create a table
    let createQuery = "create table testing(num int, name varchar);"
    let createPS = try PreparedStatement(SQLQuery: createQuery, SQLightDB: db)
    
    // executes the query
    rowsChanged = try createPS.modify()
    
    //  always destroy the query after its use
    try createPS.destroy()
    /**************************************/
    
    
    /**************************************/
    // insert values using already constructed string
    let insertQuery = "insert into testing values(12,'anything');"
    let insertPS = try PreparedStatement(SQLQuery: insertQuery, SQLightDB: db)
    
    // executes the query
    rowsChanged = try insertPS.modify()
    
    //  always destroy the query after its use
    try insertPS.destroy()
    /**************************************/
    
    
    /**************************************/
    // Preparing statement is costly, so preparing every insert statement is not recommened and prone to sql injections
    // For performance and security, parameter binding is recommened.
    let bindQuery = "insert into testing values(@num, @name);"
    let bindPS = try PreparedStatement(SQLQuery: bindQuery, SQLightDB: db)
    
    // Binding can be done in 3 ways
    for i in 0...5 // -> 1st
    {
        try bindPS.bindValues([i,"jon"]) // takes [Any]. Binds values from left to right
        rowsChanged = try bindPS.modify() // execute
        bindPS.resetBindValues() // removes the binded values for next use
    }
    
    for i in 0...5 // -> 2nd
    {
        try bindPS.bindValues([1:i,2:"john"]) // takes [Int:Any]. key -> parameter index, value -> value to bind
        rowsChanged = try bindPS.modify() // execute
        bindPS.resetBindValues() // removes the binded values for next use
    }
    
    for i in 0...5 // -> 3rd
    {
        try bindPS.bindValues(["@num":i,"@name":"nothing"]) // takes [String:Any]. key -> parameter name, value -> value to bind
        rowsChanged = try bindPS.modify() // execute
        bindPS.resetBindValues() // removes the binded values for next use
    }
    
    // You can get the number of bind parameters
    print("Number of bind parameters: ",bindPS.getParameterCount())
    
    // You can get the name of the parameters by passing index
    print("Parameter name of index 1: ",bindPS.getParameterName(parameterIndex: 1)!)
    print("Parameter name of index 2: ",bindPS.getParameterName(parameterIndex: 2)!)
    
    // You can get the index of a parameter by passing its name
    print("Parameter index of name \"@num:\" ",bindPS.getParameterIndex(parameterName: "@num"))
    print("Parameter index of name \"@name:\" ",bindPS.getParameterIndex(parameterName: "@name"))
    
    //  always destroy the query after its use
    try bindPS.destroy()
    /**************************************/
    
    
    /**************************************/
    let selectQuery = "select *from testing;"
    let selectPS = try PreparedStatement(SQLQuery: selectQuery, SQLightDB: db)
    
    // If the data is small enough to store in RAM
    let result = try selectPS.fetchAllRows() // returns [[Any]]
    for i in result
    {
        for j in i
        {
            print(j, terminator: " ")
        }
        print("")
    }
    
    // If the data is large, retrieve 1 row at a time
    while true
    {
        if let row = try selectPS.fetchNextRow() // returns [Any]?
        {
            print(row)
        }
        else
        {
            break
        }
    }
    
    // Or use a callback. The closure is called for every row. But parameter bindings is not supported
    try selectPS.execute(callbackForSelectQuery: {
        void, columnCount, values, columns in
        
        for i in 0 ..< Int(columnCount)
        {
            guard let value = values?[i]
                else { continue }
            guard let column = columns?[i]
                else { continue }
            
            let strCol = String(cString:column) // column name as string
            let strVal = String(cString:value) // value as string
            print(strVal, terminator: " ")
        }
        print("")
        
        return 0
    })
    
    // Get columns used in select query
    if let columnNames = selectPS.getColumnNames() // returns [String]?
    {
        print("Column Names: ", columnNames)
    }
    
    //  always destroy the query after its use
    try selectPS.destroy()
    /**************************************/
    
    
    /**************************************/
    // If you want to execute multiple queries where parameter binding is not required.
    // Multiple queries as String
    let customQuery = "create table test(num int);insert into test values(10);insert into test values(11);insert into test values(12);insert into test values(12);"
    let customPS = try PreparedStatement(SQLQuery: customQuery, SQLightDB: db)
    
    // The callback is only required if there is any select query
    try customPS.execute(callbackForSelectQuery: nil)
    
    //  always destroy the query after its use
    try customPS.destroy()
    /**************************************/
    
    
    /**************************************/
    // close the database connection
    try db.close()
    /**************************************/
}
catch let error as DBError
{
    print("Error message: ",error.message, "\nError code: ",error.errorCode)
}

```
>Note: Currently it doesn't support blob data type.


## Installation

#### For macOS:
1) Including Source files.
2) Including Framework.

##### Including Source Files:

1) Copy `SQLight.swift`, `PreparedStatement.swift`, `CustomTypes.swift` to your project.
2) Create a Bridging Header and add line `#include<sqlite3.h>` in it.
3) Add `libsqlite3.tbd` framework to your project under **Linked Frameworks and Libraries**.

##### Including Framework:

1) Open the SQLight Xcode project and build the project for framework.
2) Add the framework to your project.

#### For iOS:

##### Including Source Files: (same as macOS)

1) Copy `SQLight.swift`, `PreparedStatement.swift`, `CustomTypes.swift` to your project.
2) Create a Bridging Header and add line `#include<sqlite3.h>` in it.
3) Add `libsqlite3.tbd` framework to your project under **Linked Frameworks and Libraries**.


## Contributing

Feel free to fork the project and submit a pull request with your changes!

##### Not experienced or lazy to fork and submit a pull request ?
Open an issue for adding new features, enhancement, bugs etc.
 
License
----

MIT


**Free Software, Hell Yeah!**

