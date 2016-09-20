### アプリケーションの自動復旧

* [IX. 廃棄容易性](https://12factor.net/ja/disposability)


`src/main/java/mrs/app/login/LoginController.java`

``` diff
+	@GetMapping("kill")
+	void kill() {
+		System.exit(-1);
+	}
```

`src/main/java/mrs/WebSecutiryConfig.java`

``` diff
 	@Override
  	protected void configure(HttpSecurity http) throws Exception {
  		http.authorizeRequests()		  		
-				.antMatchers("/js/**", "/css/**").permitAll().antMatchers("/**")		 
+				.antMatchers("/js/**", "/css/**", "/kill").permitAll().antMatchers("/**")
  				.authenticated().and().formLogin().loginPage("/loginForm")
  				.loginProcessingUrl("/login").usernameParameter("username")
  				.passwordParameter("password").defaultSuccessUrl("/rooms", true)
```



```
./mvnw clean package -DskipTests=true
cf push
```
