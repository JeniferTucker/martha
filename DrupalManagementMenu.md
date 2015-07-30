There are duplicate menu items appearing in the administration list:

Content (mlid 5)
Content (mlid 14)
Content (mlid 897)

```
mysql> select menu_name,mlid,plid,link_title,has_children,hidden from menu_links where link_title='Content' and menu_name="management";
+------------+------+------+------------+--------------+--------+
| menu_name  | mlid | plid | link_title | has_children | hidden |
+------------+------+------+------------+--------------+--------+
| management |    5 |    3 | Content    |            0 |      0 |
| management |   14 |    5 | Content    |            0 |     -1 |
| management |  897 |    3 | Content    |            0 |      0 |
+------------+------+------+------------+--------------+--------+
3 rows in set (0.01 sec)
```

mlid 14 appears already disabled (not shown in menu display)
www.example.com/admin/structure/menu/manage/management

Disable mlid 897 for now.

```
+------------+------+------+------------+--------------+--------+
| menu_name  | mlid | plid | link_title | has_children | hidden |
+------------+------+------+------------+--------------+--------+
| management |    5 |    3 | Content    |            0 |      0 |
| management |   14 |    5 | Content    |            0 |     -1 |
| management |  897 |    3 | Content    |            0 |     -1 |
+------------+------+------+------------+--------------+--------+
3 rows in set (0.01 sec)
```

```
mysql> select menu_name,mlid,plid,link_title,has_children,hidden from menu_links where plid=3 and menu_name="management";
+------------+------+------+---------------+--------------+--------+
| menu_name  | mlid | plid | link_title    | has_children | hidden |
+------------+------+------+---------------+--------------+--------+
| management |    5 |    3 | Content       |            0 |      0 |
| management |  988 |    3 | Modules       |            0 |      0 |
| management |  989 |    3 | People        |            0 |      0 |
| management |  983 |    3 | Appearance    |            0 |      0 |
| management |  897 |    3 | Content       |            0 |      1 |
| management | 1685 |    3 | Dashboard     |            0 |      0 |
| management |  354 |    3 | Reports       |            1 |      0 |
| management |  991 |    3 | Structure     |            1 |      0 |
| management |  984 |    3 | Configuration |            1 |      0 |
| management | 1633 |    3 | Help          |            1 |      0 |
+------------+------+------+---------------+--------------+--------+
10 rows in set (0.06 sec)

mysql> 
mysql> 
mysql> select menu_name,mlid,plid,link_title,has_children,hidden from menu_links where plid=5 and menu_name="management";
+------------+------+------+------------+--------------+--------+
| menu_name  | mlid | plid | link_title | has_children | hidden |
+------------+------+------+------------+--------------+--------+
| management |   14 |    5 | Content    |            0 |     -1 |
| management | 1113 |    5 | Webforms   |            0 |     -1 |
+------------+------+------+------------+--------------+--------+
```

```
mysql> describe menu_links;
+--------------+------------------+------+-----+---------+----------------+
| Field        | Type             | Null | Key | Default | Extra          |
+--------------+------------------+------+-----+---------+----------------+
| menu_name    | varchar(32)      | NO   | MUL |         |                |
| mlid         | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| plid         | int(10) unsigned | NO   |     | 0       |                |
| link_path    | varchar(255)     | NO   | MUL |         |                |
| router_path  | varchar(255)     | NO   | MUL |         |                |
| link_title   | varchar(255)     | NO   |     |         |                |
| options      | blob             | YES  |     | NULL    |                |
| module       | varchar(255)     | NO   |     | system  |                |
| hidden       | smallint(6)      | NO   |     | 0       |                |
| external     | smallint(6)      | NO   |     | 0       |                |
| has_children | smallint(6)      | NO   |     | 0       |                |
| expanded     | smallint(6)      | NO   |     | 0       |                |
| weight       | int(11)          | NO   |     | 0       |                |
| depth        | smallint(6)      | NO   |     | 0       |                |
| customized   | smallint(6)      | NO   |     | 0       |                |
| p1           | int(10) unsigned | NO   |     | 0       |                |
| p2           | int(10) unsigned | NO   |     | 0       |                |
| p3           | int(10) unsigned | NO   |     | 0       |                |
| p4           | int(10) unsigned | NO   |     | 0       |                |
| p5           | int(10) unsigned | NO   |     | 0       |                |
| p6           | int(10) unsigned | NO   |     | 0       |                |
| p7           | int(10) unsigned | NO   |     | 0       |                |
| p8           | int(10) unsigned | NO   |     | 0       |                |
| p9           | int(10) unsigned | NO   |     | 0       |                |
| updated      | smallint(6)      | NO   |     | 0       |                |
+--------------+------------------+------+-----+---------+----------------+
25 rows in set (0.03 sec)

```

Where is the plid (parent link?) stored?

Do not delete until plid is located to see what side effects might be!

```
mysql> select menu_name,mlid,plid,link_title,has_children,hidden from menu_links where mlid=14;
+------------+------+------+------------+--------------+--------+
| menu_name  | mlid | plid | link_title | has_children | hidden |
+------------+------+------+------------+--------------+--------+
| management |   14 |    5 | Content    |            0 |      0 |
+------------+------+------+------------+--------------+--------+
1 row in set (0.00 sec)

mysql> delete from menu_links where mlid=14;
```

```
mysql> select menu_name,mlid,plid,link_title,has_children,hidden from menu_links where plid=3;
+------------+------+------+---------------+--------------+--------+
| menu_name  | mlid | plid | link_title    | has_children | hidden |
+------------+------+------+---------------+--------------+--------+
| management |    5 |    3 | Content       |            0 |      0 |  <
| management |  354 |    3 | Reports       |            1 |      0 |
| management |  991 |    3 | Structure     |            1 |      0 |
| management |  988 |    3 | Modules       |            0 |      0 |
| management |  989 |    3 | People        |            0 |      0 |
| management |  984 |    3 | Configuration |            1 |      0 |
| management |  983 |    3 | Appearance    |            0 |      0 |
| management |  897 |    3 | Content       |            0 |      0 |  <
| management | 1633 |    3 | Help          |            1 |      0 |
| management | 1685 |    3 | Dashboard     |            0 |      0 |
+------------+------+------+---------------+--------------+--------+
10 rows in set (0.01 sec)
```

Shows 2 'Content' link titles with the same plid.

```
mysql> delete from menu_links where mlid=897;
```