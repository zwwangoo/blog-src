
创建数据库表
在 gsql 提示符下，你可以使用 CREATE TABLE 语句来创建表。以下是一个示例，创建一个名为 employees 的表：

```
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    position VARCHAR(100),
    salary NUMERIC(10, 2)
);

```const targetPath = `${blogSrcPath}/${file.name}`;