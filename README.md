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

আপনি কি আপনার Laravel Tenancy project, নাকি ABP Framework-এর multi-tenant application-এর জন্য এই MySQL database তৈরি করছেন? সেই অনুযায়ী পরবর্তী configuration দেব।
