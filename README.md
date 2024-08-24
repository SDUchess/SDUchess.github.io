

# 项目开发说明文档

## 数据库指南

此文档旨在提供有关 `Chess Application` 项目中数据库结构的详细说明。数据库由四个主要表组成，分别是 `users`、`teacher_student`、`chessboards` 和 `chess_moves`。每个表都代表了应用程序中的不同实体，并具有特定的功能。

### 1. 表结构

#### 1.1 `users` 表

`users` 表存储了所有用户的基本信息，包括他们的用户名、密码和角色（教师或学生）。

- **表名**: `users`
- **描述**: 用户表，用于存储用户信息（学生和教师）。

| 字段名   | 数据类型     | 约束                        | 描述                               |
| -------- | ------------ | --------------------------- | ---------------------------------- |
| id       | BIGINT(20)   | PRIMARY KEY, AUTO_INCREMENT | 用户的唯一标识符                   |
| username | VARCHAR(255) | NOT NULL, UNIQUE            | 用户名                             |
| password | VARCHAR(255) | NOT NULL                    | 用户密码                           |
| role     | VARCHAR(50)  | NOT NULL                    | 用户角色（`student` 或 `teacher`） |

#### 1.2 `teacher_student` 表

`teacher_student` 表是一个关联表，用于存储教师和学生之间的关系。每一行记录了一个教师和一个学生之间的关系。

- **表名**: `teacher_student`
- **描述**: 教师和学生之间的关联表。

| 字段名     | 数据类型   | 约束                        | 描述                   |
| ---------- | ---------- | --------------------------- | ---------------------- |
| id         | BIGINT(20) | PRIMARY KEY, AUTO_INCREMENT | 记录的唯一标识符       |
| teacher_id | BIGINT(20) | FOREIGN KEY, NOT NULL       | 关联的教师的唯一标识符 |
| student_id | BIGINT(20) | FOREIGN KEY, NOT NULL       | 关联的学生的唯一标识符 |

- **外键约束**:
  - `teacher_id` 参考 `users(id)`，其中 `users.role` 应为 `teacher`。
  - `student_id` 参考 `users(id)`，其中 `users.role` 应为 `student`。

#### 1.3 `chessboards` 表

`chessboards` 表存储了教师创建的所有残局的基本信息，包括名称、初始棋盘布局和关联的教师。

- **表名**: `chessboards`
- **描述**: 存储残局信息的表。

| 字段名        | 数据类型     | 约束                        | 描述                             |
| ------------- | ------------ | --------------------------- | -------------------------------- |
| id            | BIGINT(20)   | PRIMARY KEY, AUTO_INCREMENT | 棋盘的唯一标识符                 |
| name          | VARCHAR(255) | NOT NULL                    | 残局的名称                       |
| initial_board | TEXT         | NOT NULL                    | 初始棋盘布局，以 JSON 字符串存储 |
| teacher_id    | BIGINT(20)   | FOREIGN KEY, NOT NULL       | 关联的教师的唯一标识符           |

- **外键约束**:
  - `teacher_id` 参考 `users(id)`，其中 `users.role` 应为 `teacher`。

#### 1.4 `chess_moves` 表

`chess_moves` 表存储了每个残局中的棋子移动信息。每一行记录了一个具体的移动步骤，包括移动顺序和具体的移动内容。

- **表名**: `chess_moves`
- **描述**: 存储每个残局中的棋子移动步骤。

| 字段名        | 数据类型     | 约束                        | 描述                                  |
| ------------- | ------------ | --------------------------- | ------------------------------------- |
| id            | BIGINT(20)   | PRIMARY KEY, AUTO_INCREMENT | 移动的唯一标识符                      |
| chessboard_id | BIGINT(20)   | FOREIGN KEY, NOT NULL       | 关联的棋盘的唯一标识符                |
| move_order    | INT(11)      | NOT NULL                    | 移动的顺序，从 1 开始                 |
| move          | VARCHAR(255) | NOT NULL                    | 移动的具体内容，格式为 `x1,y1->x2,y2` |

- **外键约束**:
  - `chessboard_id` 参考 `chessboards(id)`。

### 2. 数据库关系

- `users` 表和 `teacher_student` 表之间是 `一对多` 关系，一个教师可以管理多个学生，但一个学生只能由一个教师管理。
- `chessboards` 表和 `chess_moves` 表之间是 `一对多` 关系，一个残局棋盘包含多个移动步骤。
- `users` 表和 `chessboards` 表之间是 `一对多` 关系，一个教师可以创建多个残局。

### 3. 建表 SQL 语句示例

以下是为上述表结构编写的 SQL 语句示例：

```sql
CREATE TABLE users (
    id BIGINT(20) PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL
);

CREATE TABLE teacher_student (
    id BIGINT(20) PRIMARY KEY AUTO_INCREMENT,
    teacher_id BIGINT(20) NOT NULL,
    student_id BIGINT(20) NOT NULL,
    FOREIGN KEY (teacher_id) REFERENCES users(id),
    FOREIGN KEY (student_id) REFERENCES users(id)
);

CREATE TABLE chessboards (
    id BIGINT(20) PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    initial_board TEXT NOT NULL,
    teacher_id BIGINT(20) NOT NULL,
    FOREIGN KEY (teacher_id) REFERENCES users(id)
);

CREATE TABLE chess_moves (
    id BIGINT(20) PRIMARY KEY AUTO_INCREMENT,
    chessboard_id BIGINT(20) NOT NULL,
    move_order INT(11) NOT NULL,
    move VARCHAR(255) NOT NULL,
    FOREIGN KEY (chessboard_id) REFERENCES chessboards(id)
);
```

## 后端 API 接口文档

### 1. 用户管理 API (`/api/users`)

#### 1.1 注册用户

- **路径**: `/api/users/register`

- **方法**: `POST`

- **描述**: 注册新用户。

- **请求体**:

  ```json
  {
    "username": "string",
    "password": "string",
    "role": "student or teacher"
  }
  ```

- **响应**: 返回注册成功的用户信息。

- **错误**:

  - 用户名已存在: 返回 400 状态码，错误信息 `用户名已存在`。

#### 1.2 用户登录

- **路径**: `/api/users/login`

- **方法**: `POST`

- **描述**: 用户登录。

- **请求体**:

  ```json
  {
    "username": "string",
    "password": "string"
  }
  ```

- **响应**: 返回登录成功的用户信息。

- **错误**:

  - 用户名或密码错误: 返回 400 状态码，错误信息 `用户名或密码错误`。

#### 1.3 获取学生列表

- **路径**: `/api/users/students`
- **方法**: `GET`
- **描述**: 获取所有学生列表，支持通过用户名搜索。
- **请求参数**:
  - `username` (可选): 根据用户名模糊搜索学生。
- **响应**: 返回符合条件的学生列表。
- **示例**:
  - `/api/users/students?username=John`

#### 1.4 添加学生到教师的管理列表

- **路径**: `/api/users/addStudentToTeacher`
- **方法**: `POST`
- **描述**: 将学生添加到教师的管理列表中。
- **请求参数**:
  - `teacherId` (必填): 教师的 ID。
  - `studentId` (必填): 学生的 ID。
- **响应**: 成功时返回状态码 200，消息 `学生已成功加入管理列表`。
- **错误**:
  - 添加失败: 返回 500 状态码，错误信息 `添加学生到管理列表失败`。

#### 1.5 获取教师管理的学生列表

- **路径**: `/api/users/teacher/{teacherId}/students`
- **方法**: `GET`
- **描述**: 根据教师 ID 获取其管理的所有学生。
- **路径参数**:
  - `teacherId` (必填): 教师的 ID。
- **响应**: 返回该教师管理的所有学生列表。

#### 1.6 删除学生

- **路径**: `/api/users/teacher/{teacherId}/student/{studentId}`
- **方法**: `DELETE`
- **描述**: 从教师的管理列表中删除学生。
- **路径参数**:
  - `teacherId` (必填): 教师的 ID。
  - `studentId` (必填): 学生的 ID。
- **响应**: 成功时返回状态码 200，消息 `学生已成功从管理列表中删除`。
- **错误**:
  - 删除失败: 返回 500 状态码，错误信息 `删除学生失败`。

### 2. 残局管理 API (`/api/chessboard`)

#### 2.1 保存残局

- **路径**: `/api/chessboard/save`

- **方法**: `POST`

- **描述**: 保存一个新的残局棋盘。

- **请求体**:

  ```json
  {
    "name": "string",
    "initialBoard": "JSON string",
    "teacherId": "long"
  }
  ```

- **响应**: 返回保存成功的残局棋盘信息。

#### 2.2 保存残局的棋子移动记录

- **路径**: `/api/chessboard/moves/save`

- **方法**: `POST`

- **描述**: 保存一个残局的棋子移动记录。

- **请求体**:

  ```json
  [
    {
      "chessboardId": "long",
      "moveOrder": "int",
      "move": "string"
    },
    ...
  ]
  ```

- **响应**: 返回保存成功的所有棋子移动记录。

#### 2.3 获取指定 ID 的残局信息

- **路径**: `/api/chessboard/{id}`
- **方法**: `GET`
- **描述**: 根据 ID 获取残局信息。
- **路径参数**:
  - `id` (必填): 残局的 ID。
- **响应**: 返回指定 ID 的残局信息，如果找不到则返回 404。

#### 2.4 获取所有残局信息

- **路径**: `/api/chessboard/all`
- **方法**: `GET`
- **描述**: 获取所有残局的信息，包括移动步数统计。
- **响应**: 返回所有残局的信息列表。

#### 2.5 获取指定残局的棋子移动记录

- **路径**: `/api/chessboard/{id}/moves`
- **方法**: `GET`
- **描述**: 获取指定 ID 的残局的所有棋子移动记录。
- **路径参数**:
  - `id` (必填): 残局的 ID。
- **响应**: 返回该残局的所有棋子移动记录。

#### 2.6 删除残局

- **路径**: `/api/chessboard/{id}`
- **方法**: `DELETE`
- **描述**: 删除指定 ID 的残局。
- **路径参数**:
  - `id` (必填): 残局的 ID。
- **响应**: 无内容，成功删除残局及其所有相关记录。

#### 2.7 获取学生可访问的残局列表

- **路径**: `/api/chessboard/student/{studentId}`
- **方法**: `GET`
- **描述**: 获取学生可访问的所有残局。
- **路径参数**:
  - `studentId` (必填): 学生的 ID。
- **响应**: 返回该学生可访问的所有残局列表。
