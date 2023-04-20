# 使用说明

### 下载JAR包

从这里下载合适的[JAR包](https://jdbc.postgresql.org/download.html)，并加入IDEA的路径中。

### 连接数据库

```java
/**
     * @method getConn() 获取数据库的连接
     * @return Connection
     */
    public Connection getConn() {
        String driver = "org.postgresql.Driver";
        String url = "jdbc:postgresql://localhost:5432/spj";
        String username = "postgres";
        String password = "postgres";
        Connection conn = null;
        try {
            Class.forName(driver); // classLoader,加载对应驱动
            conn = DriverManager.getConnection(url, username, password);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return conn;
```

# CRUD

- 在进行CRUD中，一种方法是将变量和SQL语句写在一起组成String直接执行，另一种更好的方法是使用`PreparedStatement`。
- `PreparedStatement`是预编译的，并且直观容易修改，对于批量处理可以大大提高效率。

## Update

```java
/**
     * @method update(Student student) 更改表中数据
     * @return int 成功更改表中数据条数
     */
    public int update(S s) {
        Connection conn = getConn();
        int i = 0;
        String sql = "update S set sname= ? where sno=?  ";
        PreparedStatement pstmt;
        try {
            pstmt = (PreparedStatement) conn.prepareStatement(sql);
            pstmt.setString(1,s.getSname());
            pstmt.setString(2,s.getSno());

            i = pstmt.executeUpdate();
            System.out.println("resutl: " + i);
            pstmt.close();
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return i;
    }
```

## Delete

```java
/**
     * @method delete(Student student) 删除表中数据
     * @return int 成功删除表中数据条数
     */
    public int delete(String no) {
        Connection conn = getConn();
        int i = 0;
        String sql = "delete from S where sno=?";
        PreparedStatement pstmt;
        try {
            pstmt = (PreparedStatement) conn.prepareStatement(sql);
            pstmt.setString(1,no);
            i = pstmt.executeUpdate();
            System.out.println("resutl: " + i);
            pstmt.close();
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return i;
    }
```

## Insert

```java
/**
     * @method insert(Student student) 往表中插入数据
     * @return int 成功插入数据条数
     */
    public int insert(S s) {
        Connection conn = getConn();
        int i = 0;
        String sql = "insert into s (sno,sname,status,city) values(?,?,?,?)";
        PreparedStatement pstmt;
        try {
            pstmt = (PreparedStatement) conn.prepareStatement(sql);
            pstmt.setString(1, s.getSno());
            pstmt.setString(2, s.getSname());
            pstmt.setString(3, s.getStatus());
            pstmt.setString(4, s.getCity());
            i = pstmt.executeUpdate();
            pstmt.close();
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return i;
    }
```

## Select

```java
 /**
     *
     * @method Integer SelectCity(String City) 查询城市
     * @return Integer 查询并打印表中数据
     */
    public void SelectCity(String City){
        Connection conn = getConn();
        String sql = "select * from S where city = ? ";
        PreparedStatement pstmt;
        try {
            pstmt = (PreparedStatement) conn.prepareStatement(sql);
            pstmt.setString(1,City);

            ResultSet rs = pstmt.executeQuery();
            int col = rs.getMetaData().getColumnCount();
            System.out.println("============================");
//            打印每一列
            while (rs.next()) {
                for (int i = 1; i <= col; i++) {
                    System.out.print(rs.getString(i) + "\t");
                    if ((i == 2) && (rs.getString(i).length() < 8)) {
                        System.out.print("\t");
                    }
                }
                System.out.println("");
            }
            pstmt.close();
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        }
```
