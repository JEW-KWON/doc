```java
@Controller
@RequiredArgsConstructor
public class PasswordCheckController {

    private final PasswordEncoder passwordEncoder;

    @PostMapping("/password-check")
    @ResponseBody
    public boolean checkPassword(@RequestBody PasswordCheckRequest request) {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();

        return passwordEncoder.matches(request.getPassword(), userDetails.getPassword());
    }

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
```

```java