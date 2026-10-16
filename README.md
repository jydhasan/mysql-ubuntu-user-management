# mysql-ubuntu-user-management

আপনার Ubuntu WSL-এ MySQL চালানোর সময় `ERROR 1698 (28000): Access denied for user 'root'@'localhost'` দেখাচ্ছে। সাধারণত Ubuntu-তে MySQL-এর `root` user `auth_socket` authentication ব্যবহার করে। অর্থাৎ `mysql -u root -p` দিয়ে password দিলেও লগইন নাও হতে পারে; Linux-এর `sudo` দিয়ে লগইন করতে হয়।

## ধাপ ১: প্রথমে এই কমান্ডটি চালান

```
sudo mysql
```

আপনার Ubuntu user-এর password চাইলে সেটি দিন। এটি MySQL password নয়, Ubuntu/Linux user-এর password।

সফল হলে এমন prompt দেখতে পাবেন:

```
mysql>
```

## ধাপ ২: MySQL root-এর authentication যাচাই করুন

MySQL prompt-এর ভেতরে চালান:

```
SELECT user, host, plugin
FROM mysql.user
WHERE user = 'root';
```

যদি ফলাফলে `auth_socket` দেখেন, তাহলে root-এর authentication পদ্ধতিই এই সমস্যার কারণ।

## ধাপ ৩: আপনার জন্য আলাদা database user তৈরি করুন (প্রস্তাবিত)

আপনি `Tenancy` project-এ কাজ করছেন, তাই root account-এর authentication পরিবর্তন না করে project-এর জন্য আলাদা user তৈরি করা ভালো।

MySQL prompt-এ চালান:

```
CREATE USER 'tenancy_user'@'localhost'
IDENTIFIED BY 'ChangeThis_StrongPassword123!';

CREATE DATABASE tenancy_db;

GRANT ALL PRIVILEGES ON tenancy_db.*
TO 'tenancy_user'@'localhost';

EXIT;
```

`ChangeThis_StrongPassword123!`-এর জায়গায় আপনার নিজের শক্তিশালী password দিন।

## ধাপ ৪: নতুন user দিয়ে লগইন পরীক্ষা করুন

```
mysql -u tenancy_user -p tenancy_db
```

Password দিন। সফল হলে `mysql>` prompt পাবেন।

## যদি `sudo mysql`-ও কাজ না করে

প্রথমে MySQL service-এর অবস্থা দেখুন:

```
sudo service mysql status
```

বন্ধ থাকলে চালু করুন:

```
sudo service mysql start
```

তারপর আবার চেষ্টা করুন:

```
sudo mysql
```

গুরুত্বপূর্ণ: এখনই `root`-এর password reset বা authentication plugin পরিবর্তন করবেন না। আগে `sudo mysql` চালিয়ে দেখুন।

আপনি `sudo mysql` চালালে কী output আসে, সেটি এখানে পাঠান। সেই অনুযায়ী পরবর্তী সঠিক কমান্ড বলে দেব।
## ১. MySQL-এর সব user দেখুন

বর্তমান `mysql>` prompt-এ চালান:

```
SELECT user, host, plugin
FROM mysql.user;
```

এতে user-এর নাম, কোন host থেকে লগইন করতে পারবে এবং authentication plugin দেখা যাবে।

## ২. আপনার Tenancy project-এর database তৈরি করুন

প্রথমে database তৈরি করুন:

```
CREATE DATABASE tenancy_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
```

তারপর project-এর জন্য আলাদা user তৈরি করুন:

```
CREATE USER 'tenancy_user'@'localhost'
IDENTIFIED BY 'Your_Strong_Password_123!';
```

Database-এর permission দিন:

```
GRANT ALL PRIVILEGES ON tenancy_db.*
TO 'tenancy_user'@'localhost';
```

যাচাই করুন:

```
SHOW DATABASES;
SHOW GRANTS FOR 'tenancy_user'@'localhost';
```

## ৩. নতুন user দিয়ে লগইন পরীক্ষা করুন

প্রথমে MySQL থেকে বের হন:

```
EXIT;
```

এরপর Ubuntu terminal-এ চালান:

```
mysql -u tenancy_user -p tenancy_db
```

আপনার সেট করা password দিন। সফল হলে `mysql>` prompt দেখতে পাবেন।

## ৪. মনে রাখবেন

- `sudo mysql` — Ubuntu-এর administrative permission ব্যবহার করে MySQL-এ প্রবেশ করে।
- `mysql -u root -p` — MySQL root account-এর নির্ধারিত authentication পদ্ধতি ব্যবহার করে।
- `tenancy_user` — আপনার application-এর জন্য আলাদা database user।
- `tenancy_db` — আপনার Tenancy project-এর database।

আমার পরামর্শ: MySQL-এর `root` account application-এ ব্যবহার করবেন না। আলাদা user ব্যবহার করুন।

হ্যাঁ, অবশ্যই পারবেন! MySQL-এ একই database-এর জন্য একাধিক user তৈরি করতে পারবেন এবং প্রত্যেক user-কে আলাদা permission বা role দিতে পারবেন। একে Role-Based Access Control (RBAC)-এর ভিত্তি হিসেবে ব্যবহার করা যায়।

ধরুন, আপনার `tenancy_db` database-এ ৪ জন user আছে:

| User              | কাজ                    | Permission             |
| ----------------- | ---------------------- | ---------------------- |
| `tenancy_admin`   | Database administrator | সব কাজ                 |
| `tenancy_editor`  | Data entry             | SELECT, INSERT, UPDATE |
| `tenancy_auditor` | Audit / review         | শুধু SELECT            |
| `tenancy_viewer`  | শুধু দেখা              | শুধু SELECT            |

## ধাপ ১: MySQL-এ Role তৈরি করুন

আপনি বর্তমানে `tenancy_user` দিয়ে লগইন করেছেন। Role ও user তৈরি করার জন্য আপনার বর্তমান user-এর permission যথেষ্ট নয়। প্রথমে বের হয়ে root হিসেবে প্রবেশ করুন।

```
EXIT;
```

তারপর Ubuntu terminal-এ:

```
sudo mysql
```

এবার MySQL-এ নিচের SQL চালান:

```
CREATE ROLE 'tenancy_admin_role';
CREATE ROLE 'tenancy_editor_role';
CREATE ROLE 'tenancy_auditor_role';
CREATE ROLE 'tenancy_viewer_role';
```

## ধাপ ২: প্রতিটি Role-কে Permission দিন

Admin role: database-এর সব privilege।

```
GRANT ALL PRIVILEGES ON tenancy_db.*
TO 'tenancy_admin_role';
```

Editor role: data দেখা, যোগ করা ও পরিবর্তন করা।

```
GRANT SELECT, INSERT, UPDATE ON tenancy_db.*
TO 'tenancy_editor_role';
```

Auditor role: data দেখা এবং audit review করা।

```
GRANT SELECT ON tenancy_db.*
TO 'tenancy_auditor_role';
```

Viewer role: শুধু data দেখা।

```
GRANT SELECT ON tenancy_db.*
TO 'tenancy_viewer_role';
```

এখানে `DELETE`, `DROP` ও `ALTER` permission Editor-কে দেওয়া হয়নি।

## ধাপ ৩: আলাদা User তৈরি করুন

```
CREATE USER 'admin1'@'localhost'
IDENTIFIED BY 'Admin_Strong_123!';

CREATE USER 'editor1'@'localhost'
IDENTIFIED BY 'Editor_Strong_123!';

CREATE USER 'auditor1'@'localhost'
IDENTIFIED BY 'Auditor_Strong_123!';

CREATE USER 'viewer1'@'localhost'
IDENTIFIED BY 'Viewer_Strong_123!';
```

নিজের environment-এর জন্য শক্তিশালী, আলাদা password ব্যবহার করবেন।

## ধাপ ৪: User-দের Role Assign করুন

```
GRANT 'tenancy_admin_role' TO 'admin1'@'localhost';
GRANT 'tenancy_editor_role' TO 'editor1'@'localhost';
GRANT 'tenancy_auditor_role' TO 'auditor1'@'localhost';
GRANT 'tenancy_viewer_role' TO 'viewer1'@'localhost';
```

প্রতিটি user-এর জন্য default role নির্ধারণ করুন:

```
SET DEFAULT ROLE ALL TO 'admin1'@'localhost';
SET DEFAULT ROLE ALL TO 'editor1'@'localhost';
SET DEFAULT ROLE ALL TO 'auditor1'@'localhost';
SET DEFAULT ROLE ALL TO 'viewer1'@'localhost';
```

MySQL-এ `GRANT` করার পর সাধারণত `FLUSH PRIVILEGES` চালানোর প্রয়োজন হয় না।

## ধাপ ৫: Permission যাচাই করুন

```
SHOW GRANTS FOR 'auditor1'@'localhost';
SHOW GRANTS FOR 'tenancy_auditor_role';
```

Auditor-এর জন্য `SELECT` permission থাকা উচিত, কিন্তু `INSERT`, `UPDATE` বা `DELETE` থাকা উচিত নয়।

পরীক্ষা করতে:

```
mysql -u auditor1 -p tenancy_db
```

তারপর:

```
SHOW TABLES;
```

টেবিলের data দেখতে পারবেন, যদি table থাকে এবং permission ঠিকভাবে কার্যকর হয়। কিন্তু data পরিবর্তনের চেষ্টা করলে permission error পাওয়ার কথা।

## গুরুত্বপূর্ণ: Database Role বনাম Application Role

আপনার উদ্দেশ্য অনুযায়ী এই দুইটি বিষয় আলাদা।

MySQL Role

Database-এ কে query চালাতে পারবে, data পড়তে বা পরিবর্তন করতে পারবে, তা নিয়ন্ত্রণ করে।

Application Role (RBAC)

আপনার ওয়েব অ্যাপে কে Login করবে, কে Report দেখবে, কে Voucher approve করবে, কে User তৈরি করবে—এসব নিয়ন্ত্রণ করে।

আমার পরামর্শ: যদি `Tenancy` বলতে একাধিক company-র জন্য একটি application বোঝান, তাহলে প্রতিটি application user-কে আলাদা MySQL account না দিয়ে সাধারণত application-level RBAC ব্যবহার করা হয়। Database-এর connection user সীমিত permission-সহ রাখা হয় এবং application প্রতিটি company-র data আলাদাভাবে নিয়ন্ত্রণ করে।

আপনি কি একটি database-এর মধ্যে Admin, Accountant, Auditor, Viewer-এর মতো user role তৈরি করতে চান, নাকি একাধিক company-র জন্য আলাদা tenant এবং তাদের নিজস্ব user/role তৈরি করতে চান? দুটির architecture আলাদা।
