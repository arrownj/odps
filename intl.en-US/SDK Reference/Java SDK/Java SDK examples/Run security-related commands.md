---
keyword: [run security-related commands, SecurityManager.runQuery\(\)]
---

# Run security-related commands

This topic describes how to use the SecurityManager.runQuery\(\) method of MaxCompute SDK for Java to run security-related commands on the MaxCompute client.

The following operations are completed:

-   Install the IntelliJ IDEA development tool. For more information, see [Install IntelliJ IDEA](/intl.en-US/Tools and Downloads/MaxCompute Studio/Tools Installation and version history/Install IntelliJ IDEA.md).
-   Create a MaxCompute Studio project and connect it to your MaxCompute project. For more information, see [Manage project connections](/intl.en-US/Tools and Downloads/MaxCompute Studio/Manage project connections.md).
-   Add project dependencies in MaxCompute Studio.

    The SecurityManager class is contained in the odps-sdk-core package. Therefore, you must add project dependencies by using the following configuration:

    ```
    <dependency>
      <groupId>com.aliyun.odps</groupId>
      <artifactId>odps-sdk-core</artifactId>
      <version>0.29.11-oversea-public</version>
    </dependency>
    ```


You can run security-related commands by using one of the following methods:

-   Use the MaxCompute client. For more information, see the instructions in [Target users](/intl.en-US/Management/Configure security features/Target users.md) and [Security configuration of a project](/intl.en-US/Management/Configure security features/Security command list/Statements for project security configurations.md).

    The commands that start with any of the following keywords are the security-related commands of MaxCompute:

    ```
    GRANT/REVOKE ...
    SHOW  GRANTS/ACL/PACKAGE/LABEL/ROLE/PRINCIPALS
    SHOW  PRIV/PRIVILEGES
    LIST/ADD/REOVE  USERS/ROLES/TRUSTEDPROJECTS
    DROP/CREATE   ROLE
    CLEAR EXPIRED  GRANTS
    DESC/DESCRIBE   ROLE/PACKAGE
    CREATE/DELETE/DROP  PACKAGE
    ADD ... TO  PACKAGE
    REMOVE ... FROM  PACKAGE
    ALLOW/DISALLOW  PROJECT
    INSTALL/UNINSTALL  PACKAGE
    LIST/ADD/REMOVE   ACCOUNTPROVIDERS
    SET  LABLE  ...
    ```

-   Use the `SecurityManager.runQuery()` method of MaxCompute SDK for Java. For more information, see the [MaxCompute SDK for Java documentation](http://www.javadoc.io/doc/com.aliyun.odps/odps-sdk-core/0.29.11-oversea-public?spm=a2c4e.11153940.blogcont686985.22.57a97573bI8DuQ&file=0.29.11-oversea-public).

    **Note:** The security-related commands of MaxCompute are not SQL commands. Therefore, you cannot create instances to run security-related commands by using SQLTask.


1.  Run the following command to create a test table named test\_label:

    ```
    CREATE TABLE IF NOT EXISTS test_label(
     key  STRING,
     value BIGINT
    );
    ```

2.  Run the following Java code in MaxCompute Studio to run the set label command on the test\_label table. For more information about how to use MaxCompute Studio, see [Overview](/intl.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Overview.md).

    ```
    import com.aliyun.odps.Column;
    import com.aliyun.odps.Odps;
    import com.aliyun.odps.OdpsException;
    import com.aliyun.odps.OdpsType;
    import com.aliyun.odps.TableSchema;
    import com.aliyun.odps.account.Account;
    import com.aliyun.odps.account.AliyunAccount;
    import com.aliyun.odps.security.SecurityManager;
    public class test {
      public static void main(String [] args) throws OdpsException {
        try {
          // init odps
          Account account = new AliyunAccount("<your_accessid>", "<your_accesskey>");
          Odps odps = new Odps(account);
          odps.setEndpoint("http://service-corp.odps.aliyun-inc.com/api");
          odps.setDefaultProject("<your_project>");
          // set label 2 to table columns
          SecurityManager securityManager = odps.projects().get().getSecurityManager();
          String res = securityManager.runQuery("SET LABEL 2 TO TABLE test_label(key, value);", false);
          System.out.println(res);
        } catch (OdpsException e) {
          e.printStackTrace();
        }
      }
    }
    ```

3.  View the result.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/7378559951/p38175.png)

4.  Verify the result.

    After the Java code is run, run the `desc test_label` command on the MaxCompute client. In this example, the result in the following figure shows that the set label command has taken effect on the test\_label table.

    ![](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/en-US/7378559951/p38189.png)

    **Note:** For more information about security-related commands in MaxCompute, see [Authorize users](/intl.en-US/Management/Configure security features/Manage users and permissions/Authorize users.md), [Column-level access control](/intl.en-US/Management/Configure security features/Column-level access control.md), [Project security configurations](/intl.en-US/Management/Configure security features/Project security configurations.md), and [Package-based resource sharing across projects](/intl.en-US/Management/Configure security features/Resource share across project space/Package-based resource sharing across projects.md).


