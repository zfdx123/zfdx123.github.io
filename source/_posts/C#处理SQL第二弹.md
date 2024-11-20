---
layout: post
title: C# 处理 SQL Server 第二弹
date: 2024-11-20 22:10:19
tags: 
    - 编程
    - C#
categories: 
    - 编程
    - C#
thumbnail: "images/sql_csharp/fm.PNG"
cover: "images/sql_csharp/fm.PNG"
excerpt: "public virtual async Task<List<T>> Select<T>(string query, SqlParameter[] sqlParameters) where T : new()"
license: cc_by_nc_sa
---

## 类 MDataBase
```Csharp
using System;
using System.Collections.Generic;
using System.Data;
using System.Data.SqlClient;
using System.Linq;
using System.Threading.Tasks;

namespace [XXX].DataBase
{
    public class MDataBase
    {
        private static readonly object _lock = new object();
        private static MDataBase _instance;

        protected readonly SqlConnection _connection;

        private MDataBase(string connectionString)
        {
            _connection = new SqlConnection(connectionString);
        }

		public static MDataBase GetInstance()
		{
			if (_instance == null)
			{
                throw new InvalidOperationException("未初始化数据库");
			}
			return _instance;
		}

		public static MDataBase GetInstance(string connectionString)
        {
            // Double-checked locking to ensure thread-safety
            if (_instance == null)
            {
                lock (_lock)
                {
                    if (_instance == null)
                    {
                        _instance = new MDataBase(connectionString);
                    }
                }
            }
            return _instance;
        }

		public virtual void Close()
        {
            if (_connection.State == ConnectionState.Open)
            {
                _connection.Close();
                _connection.Dispose();
            }
        }

        public virtual async Task<List<T>> Select<T>(string query, SqlParameter[] sqlParameters) where T : new()
        {
            if (_connection.State != ConnectionState.Open)
                await _connection.OpenAsync();

            using (SqlCommand sqlCommand = new SqlCommand(query, _connection))
            {
                if (sqlParameters != null)
                    sqlCommand.Parameters.AddRange(sqlParameters);

                using (SqlDataAdapter sqlDataAdapter = new SqlDataAdapter(sqlCommand))
                {
                    DataSet ds = new DataSet();
                    await Task.Run(() => sqlDataAdapter.Fill(ds));

                    List<T> resultList = new List<T>();
                    foreach (DataTable dataTable in ds.Tables)
                    {
                        foreach (DataRow row in dataTable.Rows)
                        {
                            T obj = new T();
                            foreach (DataColumn column in dataTable.Columns)
                            {
                                var property = typeof(T).GetProperty(column.ColumnName);
                                if (property != null && row[column] != DBNull.Value)
                                {
                                    var value = row[column];
                                    if (property.PropertyType == typeof(DateTime?))
                                    {
                                        property.SetValue(obj, (DateTime?)value);
                                    }
                                    else
                                    {
                                        property.SetValue(obj, Convert.ChangeType(value, property.PropertyType));
                                    }
                                }
                            }
                            resultList.Add(obj);
                        }
                    }
                    return resultList;
                }
            }
        }

        public virtual async Task<bool> Insert<T>(T item, string insertQuery) where T : new()
        {
            return await ExecuteNonQueryAsync(insertQuery, CreateSqlParameters(item)) > 0;
        }

        public virtual async Task<bool> Update<T>(T item, string updateQuery) where T : new()
        {
            return await ExecuteNonQueryAsync(updateQuery, CreateSqlParameters(item)) > 0;
        }

        public virtual async Task<bool> Delete<T>(T item, string deleteQuery) where T : new()
        {
            return await ExecuteNonQueryAsync(deleteQuery, CreateSqlParameters(item)) > 0;
        }

        public virtual async Task<object> ExecuteScalar(string query, SqlParameter[] parameters)
        {
            if (_connection.State != ConnectionState.Open)
                await _connection.OpenAsync();

            using (SqlCommand sqlCommand = new SqlCommand(query, _connection))
            {
                if (parameters != null)
                    sqlCommand.Parameters.AddRange(parameters);

                return await sqlCommand.ExecuteScalarAsync();
            }
        }

        public virtual async Task<int> ExecuteNonQueryAsync(string query, SqlParameter[] parameters = null)
        {
            if (_connection.State != ConnectionState.Open)
                await _connection.OpenAsync();

            using (SqlCommand sqlCommand = new SqlCommand(query, _connection))
            {
                if (parameters != null)
                    sqlCommand.Parameters.AddRange(parameters);

                return await sqlCommand.ExecuteNonQueryAsync();
            }
        }

        public virtual async Task<bool> ExecuteTransactionAsync(params Func<SqlTransaction, Task<int>>[] operations)
        {
            if (_connection.State != ConnectionState.Open)
                await _connection.OpenAsync();

            using (SqlTransaction transaction = _connection.BeginTransaction(IsolationLevel.ReadCommitted))
            {
                try
                {
                    foreach (var operation in operations)
                    {
                        int result = await operation(transaction);
                        if (result == 0)
                        {
                            transaction.Rollback();
                            return false;
                        }
                    }

                    transaction.Commit();
                    return true;
                }
                catch
                {
                    transaction.Rollback();
                    throw;
                }
            }
        }

        public virtual SqlParameter[] CreateSqlParameters<T>(T item)
        {
            var properties = typeof(T).GetProperties();
            return properties.Select(p => new SqlParameter($"@{p.Name}", p.GetValue(item) ?? DBNull.Value)).ToArray();
        }

        public virtual async Task<int> ExecuteNonQueryAsync(string query, SqlParameter[] parameters, SqlTransaction transaction)
        {
            using (SqlCommand sqlCommand = new SqlCommand(query, _connection, transaction))
            {
                if (parameters != null)
                    sqlCommand.Parameters.AddRange(parameters);

                return await sqlCommand.ExecuteNonQueryAsync();
            }
        }
    }
}

```

## 调用代码 DEMO

```Sql
CREATE TABLE Person (
    Id INT PRIMARY KEY IDENTITY,
    Name NVARCHAR(100),
    Age INT,
    BirthDate DATETIME
);
```

```Csharp
public class Person
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
    public DateTime? BirthDate { get; set; }
}
```

```Csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Final.DataBase;

class Program
{
    static async Task Main(string[] args)
    {
        string connectionString = "Your_Connection_String_Here";
        MDataBase db = MDataBase.GetInstance(connectionString);
        // 单例模式 只获取对象
        // MDataBase db = MDataBase.GetInstance();

        try
        {
            // 插入数据
            var newPerson = new Person
            {
                Name = "Alice",
                Age = 30,
                BirthDate = new DateTime(1993, 5, 1)
            };
            string insertQuery = "INSERT INTO Person (Name, Age, BirthDate) VALUES (@Name, @Age, @BirthDate)";
            bool insertResult = await db.Insert(newPerson, insertQuery);
            Console.WriteLine($"Insert successful: {insertResult}");

            // 查询数据
            string selectQuery = "SELECT Id, Name, Age, BirthDate FROM Person";
            List<Person> persons = await db.Select<Person>(selectQuery, null);
            foreach (var person in persons)
            {
                Console.WriteLine($"Id: {person.Id}, Name: {person.Name}, Age: {person.Age}, BirthDate: {person.BirthDate}");
            }

            // 更新数据
            var updatePerson = persons[0];
            updatePerson.Age = 31;
            string updateQuery = "UPDATE Person SET Name = @Name, Age = @Age, BirthDate = @BirthDate WHERE Id = @Id";
            bool updateResult = await db.Update(updatePerson, updateQuery);
            Console.WriteLine($"Update successful: {updateResult}");

            // 删除数据
            var deletePerson = persons[0];
            string deleteQuery = "DELETE FROM Person WHERE Id = @Id";
            bool deleteResult = await db.Delete(deletePerson, deleteQuery);
            Console.WriteLine($"Delete successful: {deleteResult}");

            // 事务
            bool transactionResult = await db.ExecuteTransactionAsync(
                async (transaction) =>
                {
                    // Step 1: Insert into Person table
                    string personInsertQuery = "INSERT INTO Person (Name, Age) VALUES (@Name, @Age)";
                    var personParams = new[]
                    {
                        new SqlParameter("@Name", "John Doe"),
                        new SqlParameter("@Age", 35)
                    };
                    int personInsertResult = await db.ExecuteNonQueryAsync(personInsertQuery, personParams, transaction);

                    // Simulate failure (uncomment the next line to test rollback)
                    // throw new Exception("Simulating failure after Person insert!");

                    return personInsertResult;
                },
                async (transaction) =>
                {
                    // Step 2: Get the last inserted Person ID
                    string personIdQuery = "SELECT SCOPE_IDENTITY()";
                    object personIdResult = await db.ExecuteScalar("SELECT SCOPE_IDENTITY()", null);
                    int personId = Convert.ToInt32(personIdResult);

                    // Step 3: Insert into Address table
                    string addressInsertQuery = "INSERT INTO Address (PersonId, AddressLine) VALUES (@PersonId, @AddressLine)";
                    var addressParams = new[]
                    {
                        new SqlParameter("@PersonId", personId),
                        new SqlParameter("@AddressLine", "123 Main Street")
                    };
                    return await db.ExecuteNonQueryAsync(addressInsertQuery, addressParams, transaction);
                }
            );

            Console.WriteLine($"Transaction completed successfully: {transactionResult}");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"An error occurred: {ex.Message}");
        }
        finally
        {
            db.Close();
        }
    }
}

```