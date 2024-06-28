```java
/**
 * 비밀번호 확인 요청
 */
@RestController
@RequiredArgsConstructor
public class PasswordCheckController {

    private final PasswordEncoder passwordEncoder;

    @PostMapping("/confirm-password")
    @ResponseBody
    public ResponseEntity<?> confirmPassword(@RequestBody PasswordCheckRequest request) {
        // 비밀번호가 입력되지 않았을 경우
        if (request == null || StringUtil.isNullOrEmpty(request.getPassword())) {
            return ResponseEntity.badRequest().body("비밀번호를 입력하세요");
        }
        
        // 로그인한 사용자 정보를 가져온다.
        Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        UserDetailsImpl userDetails = SecurityHelpers.extractUserDetailsImpl(principal);
        User user = userDetails.user();

        // 입력한 비밀번호와 사용자의 비밀번호가 일치하는지 확인한다.
        boolean bool = passwordEncoder.matches(request.getPassword(), user.getPassword());
        
        // 비밀번호가 일치하지 않을 경우
        if (!bool) {
            return ResponseEntity.badRequest().body("비밀번호가 일치하지 않습니다 다시 확인하세요");
        }
        
        return ResponseEntity.ok().build();
    }

    // 비밀번호 확인 요청 DTO
    public static class PasswordCheckRequest {
        private String password;

        public String getPassword() {
            return password;
        }

        public void setPassword(String password) {
            this.password = password;
        }
    }
}

/**
 * 비밀번호 확인 요청
 * Presentation Layer
 */
@RestController
@Validated
public class PasswordCheckController {

    @SuppressWarnings("unused")
    @PostMapping("/confirm-password")
    public ResponseEntity<?> confirmPassword(@RequestParam @PasswordVerified String password) {
        return SuccessResponseUtil.buildSimpleResponseEntity();
    }
}

public class SuccessResponseUtil {
    private SuccessResponseUtil() {
        throw new UnsupportedOperationException("SuccessResponseUtil class");
    }
    public static @NotNull ResponseEntity<Void> buildSimpleResponseEntity() {
        return ResponseEntity.ok().build();
    }
}

/**
 * Custom Validation
 * Presentation Layer OR Common
 */
@Target({ElementType.FIELD, ElementType.TYPE, ElementType.PARAMETER, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PasswordVerifiedValidator.class)
@Documented
public @interface PasswordVerified {
    String message() default "비밀 번호를 확인하세요";
    String notEmptyMessage() default "비밀번호를 입력 하세요";
    String wrongPasswordMessage() default "비밀번호가 일치하지 않습니다. 다시 확인하세요";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

@Slf4j
@RequiredArgsConstructor
public class PasswordVerifiedValidator implements ConstraintValidator<PasswordVerified, String> {

    private String notEmptyMessage;
    private String wrongPasswordMessage;

    private final UserValidationService userValidationService;

    @Override
    public void initialize(@NonNull PasswordVerified constraintAnnotation) {
        this.notEmptyMessage = constraintAnnotation.notEmptyMessage();
        this.wrongPasswordMessage = constraintAnnotation.wrongPasswordMessage();
    }

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (StringUtil.isNullOrEmpty(value)) {
            ValidationUtil.addConstraintViolation(context, notEmptyMessage);
            return false;
        }
        try {
            userValidationService.verifyPassword(value);
            return true;

        } catch (InvalidPasswordException e) {
            ValidationUtil.addConstraintViolation(context, wrongPasswordMessage);
            return false;
        }
    }
}

/**
 * Application Service
 * Application Layer
 */
@Service
@RequiredArgsConstructor
@Slf4j
public class UserValidationService {
    private final UserDomainService userDomainService;
    private final AuthenticationUserService authenticationUserService;

    public void verifyPassword(String rawPassword) {
        User user = authenticationUserService.getUser();
        if (!userDomainService.checkPassword(user, rawPassword)) {
            throw new InvalidPasswordException();
        }
    }
}

/**
 * Domain Service
 * Domain Layer
 */
@Service
@RequiredArgsConstructor
public class UserDomainService {
    private final PasswordEncoder passwordEncoder;

    public boolean checkPassword(@NonNull User user, @NonNull String password) {
        return passwordEncoder.matches(password, user.getPassword());
    }
}

/**
 * Security Helpers
 * Common or Infrastructure Layer
 */
@RequiredArgsConstructor
@Service
public class AuthenticationUserService {

    public User getUser() {
        Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        UserDetailsImpl userDetails = SecurityHelpers.extractUserDetailsImpl(principal);
        return userDetails.user();
    }
}
```
### 요구 사항:

 - 직원들의 근무 시간을 계산하고, 각 직원의 총 근무 시간을 구한 후, 근무 시간이 가장 많은 직원과 그 근무 시간을 출력합니다.
 - 근무 시간 데이터는 List of Map 형식으로 제공됩니다.

### 1번 코드
```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        List<Map<String, Object>> employees = new ArrayList<>();
        
        Map<String, Object> emp1 = new HashMap<>();
        emp1.put("name", "Alice");
        emp1.put("hours", Arrays.asList(8, 7, 8, 8, 7));
        employees.add(emp1);
        
        Map<String, Object> emp2 = new HashMap<>();
        emp2.put("name", "Bob");
        emp2.put("hours", Arrays.asList(7, 8, 8, 7, 8));
        employees.add(emp2);
        
        Map<String, Object> emp3 = new HashMap<>();
        emp3.put("name", "Charlie");
        emp3.put("hours", Arrays.asList(9, 8, 8, 9, 8));
        employees.add(emp3);
        
        String topEmployee = null;
        int maxHours = 0;
        
        for (Map<String, Object> employee : employees) {
            String name = (String) employee.get("name");
            List<Integer> hours = (List<Integer>) employee.get("hours");
            int totalHours = 0;
            for (int hour : hours) {
                totalHours += hour;
            }
            if (totalHours > maxHours) {
                maxHours = totalHours;
                topEmployee = name;
            }
        }
        
        System.out.println("Top employee is " + topEmployee + " with " + maxHours + " hours");
    }
}
```
### 2번 코드
```java
public class Main {
    public static void main(String[] args) {
        List<Map<String, Object>> employees = List.of(
            Map.of("name", "Alice", "hours", List.of(8, 7, 8, 8, 7)),
            Map.of("name", "Bob", "hours", List.of(7, 8, 8, 7, 8)),
            Map.of("name", "Charlie", "hours", List.of(9, 8, 8, 9, 8))
        );
        
        Map<String, Object> topEmployee = employees.stream()
            .max(Comparator.comparingInt(emp -> ((List<Integer>) emp.get("hours")).stream().mapToInt(Integer::intValue).sum()))
            .orElseThrow();
        
        String name = (String) topEmployee.get("name");
        int totalHours = ((List<Integer>) topEmployee.get("hours")).stream().mapToInt(Integer::intValue).sum();
        
        System.out.println("Top employee is " + name + " with " + totalHours + " hours");
    }
}
```
1번 코드는 여러 변수를 사용하고 반복문을 통해 각 단계를 처리하며 비교적 복잡하게 작성됨.<br>
반면, 2번 코드는 스트림 API와 람다 표현식을 활용하여 간단하고 효율적으로 동일한 결과를 얻을 수 있게 작성


### 요구 사항:

 - 학생들의 성적을 계산하고, 각 학생의 평균 성적을 구한 후, 평균 성적이 가장 높은 학생과 그 평균 성적을 출력합니다.
 - 성적 데이터는 객체 배열로 제공됩니다.
```javascript
// 1번 코드
function calculateAverage(scores) {
    let total = 0;
    for (let i = 0; i < scores.length; i++) {
        total += scores[i];
    }
    return total / scores.length;
}

function findTopStudent(students) {
    let highestAverage = 0;
    let topStudent = null;
    for (let i = 0; i < students.length; i++) {
        let student = students[i];
        let average = calculateAverage(student.scores);
        if (average > highestAverage) {
            highestAverage = average;
            topStudent = student;
        }
    }
    return topStudent;
}

const students = [
    { name: 'Alice', scores: [90, 85, 88] },
    { name: 'Bob', scores: [80, 82, 84] },
    { name: 'Charlie', scores: [95, 90, 93] }
];

const topStudent = findTopStudent(students);
console.log(`Top student is ${topStudent.name} with an average score of ${calculateAverage(topStudent.scores)}`);

// 2번 코드
const students = [
    { name: 'Alice', scores: [90, 85, 88] },
    { name: 'Bob', scores: [80, 82, 84] },
    { name: 'Charlie', scores: [95, 90, 93] }
];

const calculateAverage = scores => scores.reduce((a, b) => a + b, 0) / scores.length;

const topStudent = students.reduce((prev, curr) =>
    calculateAverage(curr.scores) > calculateAverage(prev.scores) ? curr : prev
);

console.log(`Top student is ${topStudent.name} with an average score of ${calculateAverage(topStudent.scores)}`);
```
1번 코드는 반복문을 사용하여 각 단계를 처리하고 변수를 사용하여 결과를 저장하며 비교적 복잡하게 작성됨.<br>
반면, 2번 코드는 배열 메소드와 화살표 함수를 활용하여 간단하고 효율적으로 동일한 결과를 얻을 수 있게 작성

### 요구 사항:

 - fetch API를 사용하여 데이터를 가져오는 예제를 작성합니다.
 - fetch API를 사용하여 데이터를 가져오는 예제를 async/await를 사용하여 작성합니다.
```javascript
// fetch를 사용하여 데이터를 가져오는 예제
function fetchData() {
    fetch('https://jsonplaceholder.typicode.com/posts/1')
        .then(response => {
            if (!response.ok) {
                throw new Error('Network response was not ok');
            }
            return response.json();
        })
        .then(data => {
            console.log(data);
        })
        .catch(error => {
            console.error('There was a problem with the fetch operation:', error);
        });
}

// fetchData 함수 호출
fetchData();

// fetch를 사용하여 데이터를 가져오는 예제
async function fetchDataAsync() {
    try {
        const response = await fetch('https://jsonplaceholder.typicode.com/posts/1');
        if (!response.ok) {
            throw new Error('Network response was not ok');
        }
        const data = await response.json();
        console.log(data);
    } catch (error) {
        console.error('There was a problem with the fetch operation:', error);
    }
}

// fetchDataAsync 함수 호출
fetchDataAsync();
```