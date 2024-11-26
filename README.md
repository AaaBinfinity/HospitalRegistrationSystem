# HospitalRegistrationSystem
JavaWeb课程设计20241126

# 设计目标

构建一套基于Spring boot和Vue的挂号系统

# 思路

## 患者实现查询科室进行挂号：

## 医生实现对挂号的查看：

## 管理员权限实现对科室和医生的管理功能：

好的，下面是更加详细的技术栈模块描述：

------

## 技术栈

### 1. **数据库：MySQL**

- **数据库名称**：`Registrationdb`

- **数据库配置**：

  - **数据库主机**：`localhost`
  - **数据库用户名**：`root`
  - **数据库密码**：`Cb050328`

  ```sql
  -- 创建数据库
  CREATE DATABASE IF NOT EXISTS Registrationdb;
  
  -- 使用数据库
  USE Registrationdb;
  
  -- 创建科室表
  CREATE TABLE IF NOT EXISTS department (
      department_id INT AUTO_INCREMENT PRIMARY KEY,  -- 科室ID
      name VARCHAR(50) NOT NULL,                     -- 科室名称
      description TEXT                               -- 科室描述
  );
  
  -- 创建医生表，新增密码字段和照片字段
  CREATE TABLE IF NOT EXISTS doctor (
      doctor_id INT AUTO_INCREMENT PRIMARY KEY,  -- 医生ID
      name VARCHAR(50) NOT NULL,                 -- 医生姓名
      gender VARCHAR(10),                        -- 性别
      department_id INT,                         -- 科室ID
      title VARCHAR(50),                         -- 职称
      contact VARCHAR(20),                       -- 联系方式
      password VARCHAR(255) NOT NULL,            -- 医生登录密码（存储加密后的密码）
      photo_url VARCHAR(255),                    -- 医生照片的URL或路径
      FOREIGN KEY (department_id) REFERENCES department(department_id)  -- 外键，关联科室表
  );
  
  -- 创建患者表，新增身份证号和密码字段
  CREATE TABLE IF NOT EXISTS patient (
      patient_id INT AUTO_INCREMENT PRIMARY KEY,  -- 患者ID
      name VARCHAR(50) NOT NULL,                  -- 患者姓名
      gender VARCHAR(10),                         -- 性别
      age INT,                                    -- 年龄
      contact VARCHAR(20),                        -- 联系方式
      address VARCHAR(100),                       -- 地址
      id_card VARCHAR(18) NOT NULL UNIQUE,        -- 身份证号，使用VARCHAR(18)来存储身份证号（中国身份证号为18位）
      password VARCHAR(255) NOT NULL             -- 患者登录密码（存储加密后的密码）
  );
  
  -- 创建挂号表
  CREATE TABLE IF NOT EXISTS registration (
      registration_id INT AUTO_INCREMENT PRIMARY KEY,  -- 挂号ID
      patient_id INT NOT NULL,                         -- 患者ID
      doctor_id INT NOT NULL,                          -- 医生ID
      department_id INT NOT NULL,                      -- 科室ID
      appointment_time DATETIME,                       -- 挂号时间
      status VARCHAR(20),                              -- 挂号状态
      FOREIGN KEY (patient_id) REFERENCES patient(patient_id),   -- 外键，关联患者表
      FOREIGN KEY (doctor_id) REFERENCES doctor(doctor_id),     -- 外键，关联医生表
      FOREIGN KEY (department_id) REFERENCES department(department_id)  -- 外键，关联科室表
  );
  
  -- 创建管理员表
  CREATE TABLE IF NOT EXISTS admin (
      admin_id INT AUTO_INCREMENT PRIMARY KEY,  -- 管理员ID
      username VARCHAR(50) NOT NULL,             -- 用户名
      password VARCHAR(255) NOT NULL            -- 密码
  );
  
  -- 插入科室数据
  INSERT INTO department (name, description) VALUES
  ('内科', '内科科室，治疗常见的内科疾病'),
  ('外科', '外科科室，专注外科手术'),
  ('儿科', '儿科科室，专为儿童治疗疾病'),
  ('妇产科', '妇产科科室，专门治疗妇科和产科疾病');
  
  -- 插入医生数据，指定医生所属科室
  INSERT INTO doctor (name, gender, department_id, title, contact, password, photo_url) VALUES
  ('张医生', '男', 1, '主任医师', '13800138000', '加密密码1', 'images/doctor1.jpg'),  -- 张医生属于内科
  ('李医生', '女', 2, '副主任医师', '13900139000', '加密密码2', 'images/doctor2.jpg'),  -- 李医生属于外科
  ('王医生', '男', 3, '主治医师', '13700137000', '加密密码3', 'images/doctor3.jpg'),  -- 王医生属于儿科
  ('赵医生', '女', 4, '主任医师', '13600136000', '加密密码4', 'images/doctor4.jpg');  -- 赵医生属于妇产科
  
  -- 插入患者数据
  INSERT INTO patient (name, gender, age, contact, address, id_card, password) VALUES
  ('李小明', '男', 30, '13912345678', '北京市朝阳区', '110101199001012345', '加密密码5'),
  ('王丽丽', '女', 25, '13898765432', '上海市浦东新区', '310101199508122345', '加密密码6'),
  ('张伟', '男', 60, '13712345678', '广州白云区', '440101195601012345', '加密密码7');
  
  -- 插入挂号数据，指定患者、医生和科室
  INSERT INTO registration (patient_id, doctor_id, department_id, appointment_time, status) VALUES
  (1, 1, 1, '2024-11-26 10:00:00', '已挂号'),  -- 李小明预约张医生（内科）
  (2, 2, 2, '2024-11-26 11:00:00', '已挂号'),  -- 王丽丽预约李医生（外科）
  (3, 3, 3, '2024-11-26 14:00:00', '已挂号');  -- 张伟预约王医生（儿科）
  
  -- 插入管理员数据
  INSERT INTO admin (username, password) VALUES
  ('admin', '加密密码8'),  -- 管理员账户
  ('superadmin', '加密密码9');  -- 超级管理员账户
  
  
  ```

  

### 2. **后端：Spring Boot**

- **框架**：Spring Boot
  - **Spring Data JPA**：简化数据库操作，提供数据存取接口和自动实现功能。使用JPA（Java Persistence API）操作数据库。
  - **Spring Security**：提供认证和授权功能，保证系统的安全性，特别是对管理员和医生的权限控制。
  - **Spring Boot DevTools**：提高开发效率，自动重启、热部署等功能。
  - **Spring Validation**：用于表单数据校验，确保输入数据的合法性。
  - **Spring MVC**：用于构建RESTful API，处理请求和响应。
  - **H2数据库（开发环境）**：开发过程中使用内存数据库，快速验证功能实现。
  - **MySQL连接池（HikariCP）**：用于高效管理数据库连接。
- **后端功能模块**：
  - **患者模块**：患者信息管理、挂号、查询科室和医生信息。
  - **医生模块**：查看自己的挂号信息和患者数据。
  - **管理员模块**：管理员登录、权限控制、科室和医生的管理（增、删、改）。
  - **挂号模块**：处理患者挂号、查看科室和医生的可用情况。

### 3. **前端：Vue.js**

- 框架

  ：Vue.js（前端开发框架）

  - **Vue Router**：前端路由管理，负责页面跳转和导航。
  - **Vuex**：状态管理，用于在应用中集中管理状态数据（如登录状态、用户信息等）。
  - **Axios**：用于与后端进行数据交互，处理HTTP请求。
  - **Element UI**：UI组件库，用于快速构建响应式、现代化的用户界面。
  - **Vuelidate**：用于表单验证，确保前端输入的合法性。

- 前端页面功能

  ：

  - **登录页面**：管理员、医生和患者登录。
  - **患者模块页面**：患者查看科室、选择医生并进行挂号。
  - **医生模块页面**：医生查看和管理自己的挂号信息。
  - **管理员模块页面**：管理员管理科室和医生信息，控制系统权限。
  - **挂号记录页面**：展示患者的挂号记录以及医生的接诊情况。

### 4. **技术工具和其他**

- **Git**：版本控制工具，管理项目代码。
- **Maven**：项目管理和构建工具，自动化依赖管理。
- **Postman**：接口测试工具，用于测试后端接口的功能和数据交互。
- **IDE**：IntelliJ IDEA（后端开发）、VS Code（前端开发）。
- **Docker**（可选）：容器化部署，可以将应用和数据库部署在Docker容器中，提高部署效率。
- **Jenkins**（可选）：用于持续集成和自动化部署。

### 5. **项目部署**

- **后端部署**：使用Spring Boot打包成JAR文件，部署到服务器上。
- **前端部署**：通过Vue CLI构建项目，生成静态文件，部署到Web服务器（如Nginx）。
- **数据库部署**：MySQL数据库部署在服务器中，配置数据库连接和表结构。

------

这个版本的技术栈模块提供了更加详细的说明，涵盖了每个技术组件的功能和如何协作。希望这对你有帮助！如果需要进一步扩展某一部分，随时告诉我。
