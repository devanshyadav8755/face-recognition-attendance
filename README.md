// --- Frontend: React.js ---

// App.js import React, { useState } from 'react'; import './App.css';

function App() { const [username, setUsername] = useState(''); const [password, setPassword] = useState(''); const [role, setRole] = useState('student'); const [studentName, setStudentName] = useState(''); const [status, setStatus] = useState(''); const [loggedIn, setLoggedIn] = useState(false); const [isRegister, setIsRegister] = useState(false);

const login = async () => { try { const res = await fetch('/api/login', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ username, password }) }); const data = await res.json(); if (data.success) { setLoggedIn(true); setRole(data.role); setStatus('Login successful'); } else { setStatus('Invalid credentials'); } } catch (error) { setStatus('Login error'); } };

const register = async () => { try { const res = await fetch('/api/register', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ username, password, role }) }); const data = await res.json(); setStatus(data.message); } catch (error) { setStatus('Registration error'); } };

const markAttendance = async () => { try { const res = await fetch('/api/mark-attendance', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ name: studentName }) }); const data = await res.json(); setStatus(data.message); } catch (error) { setStatus('Error marking attendance'); } };

return ( <div className="App"> <h1>Smart Attendance System</h1> {!loggedIn ? ( <div> <input type="text" placeholder="Username" onChange={e => setUsername(e.target.value)} /> <input type="password" placeholder="Password" onChange={e => setPassword(e.target.value)} /> {isRegister && ( <select onChange={e => setRole(e.target.value)}> <option value="student">Student</option> <option value="admin">Admin</option> </select> )} <button onClick={isRegister ? register : login}>{isRegister ? 'Register' : 'Login'}</button> <button onClick={() => setIsRegister(!isRegister)}> {isRegister ? 'Switch to Login' : 'Switch to Register'} </button> </div> ) : ( <div> {role === 'student' && ( <div> <input type="text" placeholder="Enter Name" onChange={e => setStudentName(e.target.value)} /> <button onClick={markAttendance}>Mark Attendance</button> </div> )} {role === 'admin' && <p>Welcome Admin! Dashboard coming soon.</p>} </div> )} <p>{status}</p> </div> ); }

export default App;

// --- Backend: Java Spring Boot ---

// AttendanceController.java @RestController @RequestMapping("/api") public class AttendanceController {

@Autowired
private AttendanceService attendanceService;

@Autowired
private UserService userService;

@PostMapping("/mark-attendance")
public ResponseEntity<?> markAttendance(@RequestBody Map<String, String> request) {
    String name = request.get("name");
    boolean success = attendanceService.markAttendance(name);
    if (success) {
        return ResponseEntity.ok(Collections.singletonMap("message", "Attendance marked for " + name));
    } else {
        return ResponseEntity.status(400).body(Collections.singletonMap("message", "Failed to mark attendance"));
    }
}

@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody Map<String, String> request) {
    String username = request.get("username");
    String password = request.get("password");
    Optional<User> user = userService.getUser(username, password);
    if (user.isPresent()) {
        Map<String, Object> response = new HashMap<>();
        response.put("success", true);
        response.put("role", user.get().getRole());
        return ResponseEntity.ok(response);
    } else {
        return ResponseEntity.ok(Collections.singletonMap("success", false));
    }
}

@PostMapping("/register")
public ResponseEntity<?> register(@RequestBody Map<String, String> request) {
    String username = request.get("username");
    String password = request.get("password");
    String role = request.get("role");
    boolean created = userService.createUser(username, password, role);
    if (created) {
        return ResponseEntity.ok(Collections.singletonMap("message", "User registered successfully"));
    } else {
        return ResponseEntity.status(400).body(Collections.singletonMap("message", "User registration failed"));
    }
}

}

// AttendanceService.java @Service public class AttendanceService {

@Autowired
private AttendanceRepository attendanceRepository;

public boolean markAttendance(String name) {
    try {
        Attendance attendance = new Attendance();
        attendance.setName(name);
        attendance.setTimestamp(LocalDateTime.now());
        attendanceRepository.save(attendance);
        return true;
    } catch (Exception e) {
        return false;
    }
}

}

// UserService.java @Service public class UserService {

@Autowired
private UserRepository userRepository;

public Optional<User> getUser(String username, String password) {
    return userRepository.findByUsernameAndPassword(username, password);
}

public boolean createUser(String username, String password, String role) {
    try {
        User user = new User();
        user.setUsername(username);
        user.setPassword(password);
        user.setRole(role);
        userRepository.save(user);
        return true;
    } catch (Exception e) {
        return false;
    }
}

}

// User.java (Entity) @Entity public class User {

@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;

private String username;
private String password;
private String role;

public Long getId() {
    return id;
}

public void setId(Long id) {
    this.id = id;
}

public String getUsername() {
    return username;
}

public void setUsername(String username) {
    this.username = username;
}

public String getPassword() {
    return password;
}

public void setPassword(String password) {
    this.password = password;
}

public String getRole() {
    return role;
}

public void setRole(String role) {
    this.role = role;
}

}

// UserRepository.java public interface UserRepository extends JpaRepository<User, Long> { Optional<User> findByUsernameAndPassword(String username, String password); }

// application.properties spring.datasource.url=jdbc:mysql://localhost:3306/attendance_db spring.datasource.username=root spring.datasource.password=yourpassword spring.jpa.hibernate.ddl-auto=update

