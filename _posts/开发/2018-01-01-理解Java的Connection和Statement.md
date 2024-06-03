---
title: 理解 Java Connection 和 Statement
categories: 开发
toc: true
permalink: /java-connection-statement.html
---

## Statement

一个Statement对象同时只能有一个结果集在活动，就是说即使没有调用ResultSet的close()方法，只要打开第二个结果集就隐含着对上一个结果集的关闭。所以，如果你想同时对多个结果集操作,就要创建多个Statement对象，如果不需要同时操作，那么可以在一个Statement对象上顺序操作多个结果集。

所以如果要同时操作多个结果集一定要让它他绑定到不同的Statement对象上。一个connection对象可以创建任意多个Statement对象，而不需要你重新获取连结。

## Connection

在使用java开发后台应用程序的时候，如果需要使用数据库，特别是试用第三方的数据库连接池的时候，使用完PreparedStatement等一定要手动关闭，最好是将关闭的代码写到finally中，保证一定能够完成关闭。 因为第三方的数据库连接池，使用的时候，获取到Connection之后，使用完成，调用的关闭方法（close()），并没有将Connection关闭，只是放回到连接池中，如果调用的这个方法，而没有手动关闭PreparedStatement等，则这个PreparedStatement并没有关闭，这样会使得开发的程序内存急速增长，java的内存回收机制可能跟不上速度，最终造成Out of memory Error。

JDBC规范要求关闭连接的时候同时关闭任何与其关联的打开的Statement，一般情况下关闭close都没问题，但如果碰到一个不负责任的数据库连接池就不好说了。连接池重写了Connection了close方法，当调用这个方法时不关闭底层连接，而是将连接放回池中。一个负责任的连接池都应该在此时将rs和stmt物理关闭，不负责任的就可能没有将rs和stmt关闭，直接将conn丢回池中，最终可能导致OutOfMemory。在dbcp中重写的close方法里就passivate这样一个方法调用，就是用来关闭rs和stmt的。

结论：Java 7 中

    1. Connection无论是否使用连接池都必须关闭

    2. 一个PreparedStatement只能关联一个ResultSet，打开第二个ResultSet时前面的ResultSet自动关闭

    3. Connection真实关闭时，PreparedStatement和ResultSet会自动关闭

    4. 一个Connection可以关联多个PreparedStatement

    5. 一个功能的多次查询尽量使用一个Connection，可以节省连接

示例一：

    ```java
    private static void test2() throws Exception {
        String url = "jdbc:mysql://127.0.0.1:3306/S10?user=root&password=123456&characterEncoding=UTF8";

        Class.forName("com.mysql.jdbc.Driver");
        try (Connection conn = (Connection) DriverManager.getConnection(url)) {
            try (PreparedStatement pstmt = conn.prepareStatement(
                    "SELECT F01, F02, F03, F04 FROM S61.T6110 WHERE F01 = ?")) {
                pstmt.setInt(1, 2);
                try (ResultSet rs = pstmt.executeQuery()) {
                    while (rs.next()) {
                        System.out.println("\t\t" + rs.getString(1));

                        pstmt.setInt(1, 22);
                        try (ResultSet rs2 = pstmt.executeQuery()) {
                            while (rs2.next()) {
                                System.out.println("\t\t" + rs2.getString(1));
                            }
                        }

                        System.out.println("\t\t" + rs.getString(4));
                    }
                }
            }
        }
    }
    ```

    执行结果：
    斜体行报错：java.sql.SQLException: Operation not allowed after ResultSet closed

示例二：

    ```java

    private static void test3() throws Exception {
        String url = "jdbc:mysql://127.0.0.1:3306/S10?user=root&password=123456&characterEncoding=UTF8";

        Class.forName("com.mysql.jdbc.Driver");
        Connection conn = (Connection) DriverManager.getConnection(url);

        PreparedStatement pstmt = conn.prepareStatement(
                "SELECT F01, F02, F03, F04 FROM S61.T6110 WHERE F01 = ?");

        pstmt.setInt(1, 2);
        ResultSet rs = pstmt.executeQuery();

        while (rs.next()) {
            System.out.println("\t\t" + rs.getString(1));
        }

        System.out.println(pstmt.isClosed());
        System.out.println(rs.isClosed());
        conn.close();
        System.out.println(pstmt.isClosed());
        System.out.println(rs.isClosed());

    }
    ```

    执行结果：
    false
    false
    true
    true

示例三

    ```java
    private static void test() throws Exception {
        String url = "jdbc:mysql://127.0.0.1:3306/S10?user=root&password=123456&characterEncoding=UTF8";

        Class.forName("com.mysql.jdbc.Driver");
        try (Connection conn = (Connection) DriverManager.getConnection(url)) {
            try (PreparedStatement pstmt = conn.prepareStatement(
                    "SELECT F01, F02, F03, F04 FROM S10._1010 WHERE F01 = 'ALLINPAY.ALLINPAY_CERT_PATH'")) {
                try (ResultSet rs = pstmt.executeQuery()) {
                    while (rs.next()) {
                        System.out.println(rs.getString(1));
                        nestFind(conn, 1);
                        System.out.println(rs.getString(2));
                        nestFind(conn, 2);
                        System.out.println(rs.getString(3));
                        nestFind(conn, 3);
                        System.out.println(rs.getString(4));
                    }
                }
            }
        }
    }

    private static void nestFind(Connection conn, int id) throws SQLException {
        try (PreparedStatement pstmt = conn.prepareStatement(
                "SELECT F01, F02, F03, F04 FROM S11._1030 WHERE F01 = ?")) {
            pstmt.setInt(1, id);
            try (ResultSet rs = pstmt.executeQuery()) {
                while (rs.next()) {
                    System.out.println("\t" + rs.getString(1));
                    nest2Find(conn, id * 5 + 1);
                    System.out.println("\t" + rs.getString(2));
                    nest2Find(conn, id * 5 + 2);
                    System.out.println("\t" + rs.getString(3));
                    nest2Find(conn, id * 5 + 3);
                    System.out.println("\t" + rs.getString(4));
                }
            }
        }
    }

    private static void nest2Find(Connection conn, int id) throws SQLException {
        try (PreparedStatement pstmt = conn.prepareStatement(
                "SELECT F01, F02, F03, F04 FROM S61.T6110 WHERE F01 = ?")) {
            pstmt.setInt(1, id);
            try (ResultSet rs = pstmt.executeQuery()) {
                while (rs.next()) {
                    System.out.println("\t\t" + rs.getString(1));
                    System.out.println("\t\t" + rs.getString(2));
                    System.out.println("\t\t" + rs.getString(3));
                    System.out.println("\t\t" + rs.getString(4));
                }
            }
        }
    }

    ```

    执行结果：可以正常执行

## PreparedStatement 和 Statement

基于以下的原因，应该始终以 PreparedStatement 代替 Statement，任何时候都不要使用Statement。

1. 可读性和可维护性

    虽然用 PreparedStatement 来代替 Statement 会使代码多出几行，但这样的代码可读性和可维护性都比直接用Statement的代码高。

2. 性能

    PreparedStatement 语句会将 DB 的编译器编译后的执行代码缓存下来，下次调用时只要缓存还存在，就不需要再次编译，只需将参数直接传入编译过的语句执行代码中即可。而 statement 的语句中，即使是相同一操作，由于每次操作的数据不同，所以使整个语句相匹配的机会极小，几乎不太可能匹配。

3. 安全性

    使用预编译语句，传入的任何内容就不会和原来的语句发生任何匹配的关系，能够有效防止 SQL 注入。
