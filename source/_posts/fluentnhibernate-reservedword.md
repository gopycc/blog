---
title: 使用FluentNHibernate出现列“ReservedWord”不属于表 ReservedWords
date: 2015-05-27 01:06:00
categories: dotnet
tags: [nhibernate]
---

NHibernate+FluentNHibernate+MySql 运行时黄页显示下边的异常，项目中找了半天没出现过这个列的关键字。
```
[ArgumentException: 列“ReservedWord”不属于表 ReservedWords。]
   System.Data.DataRow.GetDataColumn(String columnName) +5310119
   System.Data.DataRow.get_Item(String columnName) +13
   NHibernate.Dialect.Schema.AbstractDataBaseSchema.GetReservedWords() +170
   NHibernate.Tool.hbm2ddl.SchemaMetadataUpdater.GetReservedWords(Dialect dialect, IConnectionHelper connectionHelper) +100
   NHibernate.Tool.hbm2ddl.SchemaMetadataUpdater.Update(ISessionFactory sessionFactory) +78
   NHibernate.Impl.SessionFactoryImpl..ctor(Configuration cfg, IMapping mapping, Settings settings, EventListeners listeners) +700
   NHibernate.Cfg.Configuration.BuildSessionFactory() +104
   FluentNHibernate.Cfg.FluentConfiguration.BuildSessionFactory() in c:\work\coding\fluentNhibernate\src\FluentNHibernate\Cfg\FluentConfiguration.cs:230

[FluentConfigurationException: An invalid or incomplete configuration was used while creating a SessionFactory. Check PotentialReasons collection, and InnerException for more detail.]
```

百度到下边的类似问题：

error1.  **Could not create the driver from NHibernate.Driver.MySqlDataDriver**

解决方法：在使用Nhibernate连接Mysql时报这个错，请把MySql.Data.dll文件手动 拷贝到xxx/工程文件目录/bin/Debug下 就可以解决这个问题了！
 
error2. **列“ReservedWord”不属于表 ReservedWords**
解决方法：在hibernate.cfg.xml配置文件中加入<property name="hbm2ddl.keywords">none</property>
 
最后在stackoverflow上找到对应的用FluentNHibernate配置的方法（最后一行的配置）：
```
Configuration.DefaultNameOrConnectionString = ConfigurationManager.ConnectionStrings["Default"].ConnectionString;
Configuration.Modules.AbpNHibernate().FluentConfiguration
             .Database(MySQLConfiguration.Standard.ConnectionString(Configuration.DefaultNameOrConnectionString))
             .Mappings(m => m.FluentMappings.AddFromAssembly(Assembly.GetExecutingAssembly()));
             .ExposeConfiguration(c => c.Properties.Add("hbm2ddl.keywords", "none"));
 ```
