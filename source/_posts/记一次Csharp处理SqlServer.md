---
layout: post
title: 记一次C# Sql Server数据库处理
date: 2024-05-11 10:53:19
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
using System.Linq;
using System.Text;
using System.Data;
using System.Data.SqlClient;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace SqlServerTest.DataBase
{
    public class MDataBase
    {
        protected readonly SqlConnection _connection;
        public MDataBase(string connectionString)
        {
            _connection = new SqlConnection(connectionString);
        }

        public virtual async Task<List<T>> Select<T>(string query, SqlParameter[] sqlParameters) where T : new()
        {
            try
            {
                if (_connection.State != ConnectionState.Open)
                    await _connection.OpenAsync();

                using (SqlCommand sqlCommand = new SqlCommand(query, _connection))
                {
                    sqlCommand.Parameters.AddRange(sqlParameters);
                    using (SqlDataAdapter sqlDataAdapter = new SqlDataAdapter(sqlCommand))
                    {
                        DataSet ds = new DataSet();
                        await Task.Run(() => sqlDataAdapter.Fill(ds));

                        // 创建用于存储结果的列表
                        List<T> resultList = new List<T>();

                        // 遍历 DataSet 中的每个 DataTable
                        foreach (DataTable dataTable in ds.Tables)
                        {
                            // 遍历 DataTable 中的每行数据
                            foreach (DataRow row in dataTable.Rows)
                            {
                                // 创建一个新的泛型对象
                                T obj = new T();

                                // 使用反射将每列的值赋给对象的属性
                                foreach (DataColumn column in dataTable.Columns)
                                {
                                    var property = typeof(T).GetProperty(column.ColumnName);
                                    if (property != null && row[column] != DBNull.Value)
                                    {
                                        property.SetValue(obj, Convert.ChangeType(row[column], property.PropertyType));
                                    }
                                }

                                // 将对象添加到结果列表中
                                resultList.Add(obj);
                            }
                        }

                        return resultList;
                    }
                }
            }
            catch (Exception)
            {
                // 返回异常给上层
                throw;
            }
            finally
            {
                // 确保连接在完成后关闭
                _connection.Close();
            }
        }

        public virtual async Task<bool> Insert<T>(List<T> items, string insertQuery) where T : new()
        {
            try
            {
                if (_connection.State != ConnectionState.Open)
                    await _connection.OpenAsync();

                using (SqlCommand sqlCommand = new SqlCommand(insertQuery, _connection))
                {
                    foreach (var item in items)
                    {
                        sqlCommand.Parameters.Clear();
                        var properties = typeof(T).GetProperties();
                        foreach (var property in properties)
                        {
                            sqlCommand.Parameters.AddWithValue($"@{property.Name}", property.GetValue(item) ?? DBNull.Value);
                        }

                        await sqlCommand.ExecuteNonQueryAsync();
                    }

                    return true; // 操作成功
                }
            }
            catch (Exception)
            {
                // 将异常传播给调用层
                throw;
            }
            finally
            {
                _connection.Close();
            }
        }

        public virtual async Task<bool> Update<T>(T item, string updateQuery) where T : new()
        {
            try
            {
                if (_connection.State != ConnectionState.Open)
                    await _connection.OpenAsync();

                using (SqlCommand sqlCommand = new SqlCommand(updateQuery, _connection))
                {
                    sqlCommand.Parameters.Clear();
                    var properties = typeof(T).GetProperties();
                    foreach (var property in properties)
                    {
                        sqlCommand.Parameters.AddWithValue($"@{property.Name}", property.GetValue(item) ?? DBNull.Value);
                    }

                    int rowsAffected = await sqlCommand.ExecuteNonQueryAsync();
                    return rowsAffected > 0; // 如果有行受影响，则操作成功
                }
            }
            catch (Exception)
            {
                // 将异常传播给调用层
                throw;
            }
            finally
            {
                _connection.Close();
            }
        }

        public virtual async Task<bool> Delete<T>(T item, string deleteQuery) where T : new()
        {
            try
            {
                if (_connection.State != ConnectionState.Open)
                    await _connection.OpenAsync();

                using (SqlCommand sqlCommand = new SqlCommand(deleteQuery, _connection))
                {
                    sqlCommand.Parameters.Clear();
                    var properties = typeof(T).GetProperties();
                    foreach (var property in properties)
                    {
                        sqlCommand.Parameters.AddWithValue($"@{property.Name}", property.GetValue(item) ?? DBNull.Value);
                    }

                    int rowsAffected = await sqlCommand.ExecuteNonQueryAsync();
                    return rowsAffected > 0; // 如果有行受影响，则操作成功
                }
            }
            catch (Exception)
            {
                // 将异常传播给调用层
                throw;
            }
            finally
            {
                _connection.Close();
            }
        }


    }
}

```

## 调用代码
```Csharp
using System;
using System.Collections.Generic;
using System.Data.SqlClient;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace SqlServerTest.DataBase
{

    public class Person
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public int Age { get; set; }
    }

    internal class test_Data_base
    {
        public static async void test()
        {
            string connectionString = "YourConnectionString";

            // 创建 DataBase 实例
            MDataBase dataBase = new MDataBase(connectionString);

            // 查询示例
            string selectQuery = "SELECT Id, Name, Age FROM People";
            SqlParameter[] selectParameters = new SqlParameter[] { };
            var dataSet = await dataBase.Select<Person>(selectQuery, selectParameters);
            // 这里可以对返回的 DataSet 进行处理

            // 插入示例
            var person1 = new Person { Id = 1, Name = "John", Age = 30 };
            var person2 = new Person { Id = 2, Name = "Jane", Age = 25 };
            string insertQuery = "INSERT INTO People (Id, Name, Age) VALUES (@Id, @Name, @Age)";
            bool insertSuccess = await dataBase.Insert(new List<Person> { person1, person2 }, insertQuery);
            Console.WriteLine($"Insert success: {insertSuccess}");

            // 更新示例
            var updatePerson = new Person { Id = 1, Name = "John Doe", Age = 32 };
            string updateQuery = "UPDATE People SET Name = @Name, Age = @Age WHERE Id = @Id";
            bool updateSuccess = await dataBase.Update(updatePerson, updateQuery);
            Console.WriteLine($"Update success: {updateSuccess}");

            // 删除示例
            var deletePerson = new Person { Id = 2 }; // 删除 ID 为 2 的记录
            string deleteQuery = "DELETE FROM People WHERE Id = @Id";
            bool deleteSuccess = await dataBase.Delete(deletePerson, deleteQuery);
            Console.WriteLine($"Delete success: {deleteSuccess}");

        }
    }
}

```