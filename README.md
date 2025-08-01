Hướng dẫn chi tiết xây dựng dự án Spring Boot không sử dụng quan hệ trong model entity
Dưới đây là hướng dẫn từng bước để xây dựng một ứng dụng Spring Boot sử dụng Lombok, Spring Web, JPA, và MySQL mà không định nghĩa quan hệ trong các model entity. Thay vì sử dụng các annotation như @OneToMany hoặc @ManyToOne, chúng ta sẽ lưu trữ các khóa ngoại dưới dạng các trường thông thường và xử lý các mối quan hệ thông qua truy vấn JPQL. Hướng dẫn này bao gồm tất cả các bước cần thiết để bạn có thể sao chép và triển khai dự án.
1. Thiết lập dự án Spring Boot
1.1 Tạo dự án Spring Boot mới
Sử dụng Spring Initializr để tạo một dự án mới với các phụ thuộc sau:

Spring Web: Để xây dựng ứng dụng web.
Spring Data JPA: Để tương tác với cơ sở dữ liệu thông qua JPA.
MySQL Driver: Để kết nối với MySQL.
Lombok: Để giảm mã lặp (getter, setter, constructor, v.v.).
Thymeleaf: Để render các trang HTML.

Các bước:

Truy cập Spring Initializr.
Chọn:
Project: Maven hoặc Gradle.
Language: Java.
Spring Boot: Phiên bản mới nhất (ví dụ: 3.x.x).
Dependencies: Thêm Spring Web, Spring Data JPA, MySQL Driver, Lombok, Thymeleaf.


Nhấn Generate, tải dự án về và import vào IDE (như IntelliJ IDEA hoặc Eclipse).

1.2 Cấu hình kết nối cơ sở dữ liệu
Trong file src/main/resources/application.properties, thêm cấu hình kết nối MySQL:
spring.datasource.url=jdbc:mysql://localhost:3306/your_database_name
spring.datasource.username=your_username
spring.datasource.password=your_password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

Giải thích:

Thay your_database_name, your_username, và your_password bằng thông tin thực tế của cơ sở dữ liệu MySQL của bạn.
spring.jpa.hibernate.ddl-auto=update cho phép Hibernate tự động tạo hoặc cập nhật schema dựa trên entity, phù hợp cho giai đoạn phát triển. Trong môi trường sản xuất, bạn có thể thay bằng validate hoặc none.
spring.jpa.show-sql=true hiển thị các câu lệnh SQL được thực thi, giúp debug dễ dàng hơn.

Lưu ý: Đảm bảo MySQL đã được cài đặt và database your_database_name đã được tạo. Bạn có thể chạy câu lệnh CREATE DATABASE your_database_name; trong MySQL nếu cần.
2. Định nghĩa các entity mà không có quan hệ
Dựa trên schema cơ sở dữ liệu (bảng t_student, t_course, và t_student_course), bạn cần tạo các entity tương ứng mà không sử dụng các annotation quan hệ như @OneToMany hoặc @ManyToOne. Các khóa ngoại sẽ được lưu trữ dưới dạng các trường Integer thông thường.
2.1 Entity Student
Tạo file Student.java trong package com.example.demo.entity:
package com.example.demo.entity;

import lombok.Data;
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "t_student")
@Data
public class Student {
    @Id
    private Integer id;
    private String name;
    private String className;
}

2.2 Entity Course
Tạo file Course.java trong package com.example.demo.entity:
package com.example.demo.entity;

import lombok.Data;
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "t_course")
@Data
public class Course {
    @Id
    private Integer id;
    private String name;
    private Integer duration;
}

2.3 Entity StudentCourse
Tạo file StudentCourse.java trong package com.example.demo.entity:
package com.example.demo.entity;

import lombok.Data;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import java.time.LocalDateTime;

@Entity
@Table(name = "t_student_course")
@Data
public class StudentCourse {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private Integer studentId;
    private Integer courseId;
    private LocalDateTime enrollmentDate;
}

Lưu ý:

Các trường studentId và courseId trong StudentCourse được định nghĩa như các trường Integer thông thường, không sử dụng @ManyToOne để ánh xạ quan hệ.
Annotation @Data từ Lombok tự động tạo getter, setter, constructor, và các phương thức toString, equals, hashCode, giúp giảm mã lặp.
@GeneratedValue(strategy = GenerationType.IDENTITY) được sử dụng cho trường id trong StudentCourse để tự động tăng giá trị, phù hợp với schema bảng.

3. Tạo repository với truy vấn tùy chỉnh
Để xử lý các mối quan hệ giữa các bảng mà không sử dụng quan hệ trong entity, bạn cần định nghĩa các truy vấn tùy chỉnh trong repository sử dụng JPQL hoặc SQL native.
3.1 StudentRepository
Tạo file StudentRepository.java trong package com.example.demo.repository:
package com.example.demo.repository;

import com.example.demo.entity.Student;
import org.springframework.data.jpa.repository.JpaRepository;

public interface StudentRepository extends JpaRepository<Student, Integer> {
}

3.2 CourseRepository
Tạo file CourseRepository.java trong package com.example.demo.repository:
package com.example.demo.repository;

import com.example.demo.entity.Course;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import java.util.List;

public interface CourseRepository extends JpaRepository<Course, Integer> {
    @Query("SELECT c FROM Course c WHERE c.id IN (SELECT sc.courseId FROM StudentCourse sc WHERE sc.studentId = :studentId)")
    List<Course> findCoursesByStudentId(@Param("studentId") Integer studentId);
}

3.3 StudentCourseRepository
Tạo file StudentCourseRepository.java trong package com.example.demo.repository:
package com.example.demo.repository;

import com.example.demo.entity.StudentCourse;
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.List;

public interface StudentCourseRepository extends JpaRepository<StudentCourse, Integer> {
    List<StudentCourse> findByStudentId(Integer studentId);
}

Giải thích:

CourseRepository sử dụng truy vấn JPQL để lấy danh sách các khóa học của một sinh viên dựa trên studentId.
StudentCourseRepository cung cấp phương thức findByStudentId để lấy các bản ghi StudentCourse liên quan đến một sinh viên cụ thể.
Các truy vấn này thay thế cho việc sử dụng quan hệ trong entity, cho phép bạn truy xuất dữ liệu liên quan mà không cần ánh xạ quan hệ.

4. Xây dựng Controller và View
Dựa trên yêu cầu hiển thị danh sách sinh viên cùng khóa học và thêm khóa học cho sinh viên, bạn cần tạo controller và các template Thymeleaf.
4.1 Controller
Tạo file StudentController.java trong package com.example.demo.controller:
package com.example.demo.controller;

import com.example.demo.entity.Course;
import com.example.demo.entity.Student;
import com.example.demo.entity.StudentCourse;
import com.example.demo.repository.CourseRepository;
import com.example.demo.repository.StudentCourseRepository;
import com.example.demo.repository.StudentRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import java.time.LocalDateTime;
import java.util.List;

@Controller
public class StudentController {
    @Autowired
    private StudentRepository studentRepository;
    @Autowired
    private CourseRepository courseRepository;
    @Autowired
    private StudentCourseRepository studentCourseRepository;

    @GetMapping("/students")
    public String getAllStudents(Model model) {
        List<Student> students = studentRepository.findAll();
        model.addAttribute("students", students);
        return "students";
    }

    @GetMapping("/students/{id}/courses")
    public String getCoursesForStudent(@PathVariable Integer id, Model model) {
        List<Course> courses = courseRepository.findCoursesByStudentId(id);
        model.addAttribute("studentId", id);
        model.addAttribute("courses", courses);
        return "student-courses";
    }

    @GetMapping("/students/{id}/add-course")
    public String showAddCourseForm(@PathVariable Integer id, Model model) {
        Student student = studentRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Sinh viên không tồn tại"));
        List<Course> courses = courseRepository.findAll();
        model.addAttribute("student", student);
        model.addAttribute("courses", courses);
        return "add-course";
    }

    @PostMapping("/students/{id}/add-course")
    public String addCourse(@PathVariable Integer id, @RequestParam Integer courseId) {
        StudentCourse studentCourse = new StudentCourse();
        studentCourse.setStudentId(id);
        studentCourse.setCourseId(courseId);
        studentCourse.setEnrollmentDate(LocalDateTime.now());
        studentCourseRepository.save(studentCourse);
        return "redirect:/students";
    }
}

Giải thích:

@GetMapping("/students"): Hiển thị danh sách tất cả sinh viên.
@GetMapping("/students/{id}/courses"): Hiển thị danh sách khóa học của một sinh viên cụ thể.
@GetMapping("/students/{id}/add-course"): Hiển thị form để thêm khóa học cho sinh viên.
@PostMapping("/students/{id}/add-course"): Xử lý submit form, lưu bản ghi mới vào bảng t_student_course với enrollmentDate là thời gian hiện tại.

4.2 View (Thymeleaf)
Tạo các file template trong thư mục src/main/resources/templates.
students.html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Danh sách sinh viên</title>
</head>
<body>
    <h1>Danh sách sinh viên</h1>
    <table border="1">
        <tr>
            <th>ID</th>
            <th>Tên</th>
            <th>Lớp</th>
            <th>Khóa học</th>
            <th>Thêm khóa học</th>
        </tr>
        <tr th:each="student : ${students}">
            <td th:text="${student.id}"></td>
            <td th:text="${student.name}"></td>
            <td th:text="${student.className}"></td>
            <td><a th:href="@{/students/{id}/courses(id=${student.id})}">Xem khóa học</a></td>
            <td><a th:href="@{/students/{id}/add-course(id=${student.id})}">Thêm khóa học</a></td>
        </tr>
    </table>
</body>
</html>

student-courses.html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Khóa học của sinh viên</title>
</head>
<body>
    <h1>Khóa học của sinh viên ID: <span th:text="${studentId}"></span></h1>
    <ul>
        <li th:each="course : ${courses}" th:text="${course.name}"></li>
    </ul>
    <a th:href="@{/students}">Quay lại danh sách sinh viên</a>
</body>
</html>

add-course.html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Thêm khóa học cho sinh viên</title>
</head>
<body>
    <h1>Chọn khóa học cho sinh viên: <span th:text="${student.name}"></span></h1>
    <form th:action="@{/students/{id}/add-course(id=${student.id})}" method="post">
        <select name="courseId">
            <option th:each="course : ${courses}" th:value="${course.id}" th:text="${course.name}"></option>
        </select>
        <button type="submit">Submit</button>
        <button type="button" onclick="window.location.href='/students'">Cancel</button>
    </form>
</body>
</html>

Giải thích:

students.html: Hiển thị danh sách sinh viên với các cột ID, Tên, Lớp, và liên kết để xem khóa học hoặc thêm khóa học.
student-courses.html: Hiển thị danh sách khóa học của một sinh viên cụ thể.
add-course.html: Cung cấp form với dropdown chứa danh sách khóa học từ bảng t_course, cho phép chọn và thêm khóa học cho sinh viên.

5. Xử lý dữ liệu ban đầu
Để thêm dữ liệu ban đầu vào cơ sở dữ liệu, bạn có thể sao chép các câu lệnh INSERT từ file SQL vào file src/main/resources/data.sql. Spring Boot sẽ tự động chạy file này khi khởi động ứng dụng:
insert into t_student(id, name, class_name) VALUES
    (1, 'Ho Van Cuong', 'C2201LV'),
    (2, 'Tran Tien Anh', 'C2211K'),
    (3, 'Tran Anh Kien', 'C2211K'),
    (4, 'Ngo Hoang Tung', 'C2211K');
insert into t_course(id, name, duration) VALUES
    (1, 'Java programming', 20),
    (2, 'Distributed Java programming with RMI', 30),
    (3, 'Java Swing for Desktop app', 24),
    (4, 'PHP programming', 23);
insert into t_student_course(student_id, course_id, enrollment_date) values
    (1, 1, now()),
    (1, 3, now()),
    (1, 4, now()),
    (3, 1, now()),
    (3, 3, now()),
    (3, 4, now());

Lưu ý:

Nếu cơ sở dữ liệu của bạn sử dụng tự động tăng ID (AUTO_INCREMENT), bạn có thể bỏ trường id trong các câu lệnh INSERT để MySQL tự động tạo giá trị.
Đảm bảo các bảng t_student, t_course, và t_student_course đã được tạo trong cơ sở dữ liệu trước khi chạy ứng dụng. Bạn có thể sử dụng các câu lệnh CREATE TABLE từ file SQL cung cấp:

CREATE TABLE t_student (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50),
    class_name VARCHAR(50)
);

CREATE TABLE t_course (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50),
    duration INT
);

CREATE TABLE t_student_course (
    id INT PRIMARY KEY AUTO_INCREMENT,
    student_id INT,
    course_id INT,
    enrollment_date DATETIME,
    FOREIGN KEY (student_id) REFERENCES t_student(id),
    FOREIGN KEY (course_id) REFERENCES t_course(id)
);

6. Chạy và kiểm tra ứng dụng

Khởi động ứng dụng: Chạy ứng dụng Spring Boot từ IDE hoặc sử dụng lệnh:
./mvnw spring-boot:run

(nếu sử dụng Maven) hoặc
./gradlew bootRun

(nếu sử dụng Gradle).

Truy cập các URL:

http://localhost:8080/students: Xem danh sách sinh viên.
http://localhost:8080/students/{id}/courses: Xem danh sách khóa học của một sinh viên (thay {id} bằng ID sinh viên, ví dụ: 1).
http://localhost:8080/students/{id}/add-course: Hiển thị form để thêm khóa học cho sinh viên.


Kiểm tra chức năng:

Xác minh rằng danh sách sinh viên hiển thị đúng thông tin từ bảng t_student.
Kiểm tra xem danh sách khóa học của một sinh viên có hiển thị đúng các khóa học liên quan từ bảng t_student_course.
Thử thêm một khóa học cho sinh viên và kiểm tra xem bản ghi mới có được lưu vào bảng t_student_course với enrollment_date là thời gian hiện tại hay không.



7. Lợi ích và hạn chế của cách tiếp cận
Lợi ích

Đơn giản hóa quản lý entity: Tránh được sự phức tạp của việc quản lý quan hệ JPA, đặc biệt là các vấn đề liên quan đến cascade, lazy loading, hoặc eager loading.
Kiểm soát tốt hơn: Bạn có toàn quyền kiểm soát các truy vấn, giúp dễ dàng tối ưu hóa hiệu suất trong các trường hợp cụ thể.
Phù hợp với các dự án đơn giản: Nếu dự án không yêu cầu các mối quan hệ phức tạp, cách này giúp giảm mã và dễ bảo trì.

Hạn chế

Truy vấn thủ công: Bạn phải tự viết các truy vấn JPQL hoặc SQL native, có thể phức tạp và dễ xảy ra lỗi nếu không cẩn thận.
Mất các tính năng của JPA: Không sử dụng các tính năng như cascade, lazy loading, hoặc eager loading, có thể làm tăng khối lượng công việc khi xử lý các thao tác phức tạp.
Hiệu suất cần tối ưu hóa: Các truy vấn thủ công cần được tối ưu hóa cẩn thận để tránh các vấn đề như N+1 query.

8. Bảng tóm tắt các bước triển khai



Bước
Mô tả
Chi tiết



1
Tạo dự án Spring Boot
Sử dụng Spring Initializr, thêm phụ thuộc: Spring Web, JPA, MySQL, Lombok, Thymeleaf.


2
Cấu hình database
Sửa file application.properties với URL, username, password của MySQL.


3
Tạo entity mà không có quan hệ
Định nghĩa Student, Course, StudentCourse với các trường khóa ngoại thông thường.


4
Tạo repository với truy vấn tùy chỉnh
Sử dụng JPQL để xử lý các mối quan hệ giữa các bảng.


5
Xây dựng controller và view
Tạo controller cho danh sách sinh viên, khóa học, và form thêm khóa học.


6
Thêm dữ liệu ban đầu
Sử dụng file data.sql để chèn dữ liệu mẫu.


7
Chạy và kiểm tra
Khởi động ứng dụng, kiểm tra các endpoint qua trình duyệt.


9. Tài liệu tham khảo

Spring Initializr
Spring Data JPA Documentation
Thymeleaf Documentation

10. Kết luận
Hướng dẫn này cung cấp các bước chi tiết để xây dựng một ứng dụng Spring Boot sử dụng JPA và MySQL mà không cần định nghĩa quan hệ trong các model entity. Bằng cách lưu trữ khóa ngoại dưới dạng các trường thông thường và sử dụng các truy vấn JPQL, bạn có thể thực hiện các chức năng như hiển thị danh sách sinh viên, khóa học, và thêm khóa học cho sinh viên. Cách tiếp cận này giúp đơn giản hóa việc quản lý entity nhưng đòi hỏi bạn phải cẩn thận trong việc viết và tối ưu hóa các truy vấn. Nếu bạn cần thêm hỗ trợ hoặc muốn mở rộng chức năng, hãy cho tôi biết!
