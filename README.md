# Getting started with the new Authorization server in spring boot 3.x

This aims to discover the newly released version of their OAuth server. All that is required 
to get the server up and running is defined in `SecurityConfig.class`.

We first need to configure the `HttpSecurity` object with a default configuration using
``OAuth2AuthorizationServerConfiguration`` this will register all the required endpoints
even some beans.

The following snippet is mainly used in case of the AUTHORIZATION_CODE flow, which defines a basic login form.

```java 
        http.exceptionHandling(e -> e.defaultAuthenticationEntryPointFor(
                new LoginUrlAuthenticationEntryPoint("/login"),
                new MediaTypeRequestMatcher(MediaType.TEXT_HTML)));

```

```RegisteredClientRepository``` will be our datastore for clients, we are using an in memory implementation.

```UserDetailsService``` for managing our users, could be an in memory or jdbc or even jpa implementation.

```JWKSource``` used for token related ops, namely generation, signing and verification.

Keep in mind that for a production env, you'll need to generate your keys and keep them safely, to be able 
to signe the tokens with the same key when restating the server, indeed for testing purposes it's not relevant.

### Testing

##### Using the client_credentials flow:

it's basically a http basic for clients, here's how to do it in cmd using httpie:

``` http -v -f POST :8080/oauth2/token grant_type=client_credentials scope='read openid write export profile' -a client:secret```

response:
```json
{
    "access_token": "eyJraWQiOiI2NTc3OGUxZC1mYzQzLTRhMGEtOTQ5Zi03Y2YyZDYyNDUyZDciLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJjbGllbnQiLCJhdWQiOiJjbGllbnQiLCJuYmYiOjE3MTc5NDg0MjYsInNjb3BlIjpbInJlYWQiLCJvcGVuaWQiLCJwcm9maWxlIiwid3JpdGUiLCJleHBvcnQiXSwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwIiwiZXhwIjoxNzE3OTQ4NzI2LCJpYXQiOjE3MTc5NDg0MjYsImp0aSI6IjIyMmZlZGJjLWM2YTktNDIyNC1iZGMzLTcwYTlkZDkxZTYwOSJ9.D-D0nhIlJRwAn9OSHdCT00kRFvi-1zCZUHJsSKUrNKI-W-Bl2FSHjBm4QGEQ-qn6CXTzHeleV9pGyAXq7JNLsuvI3lslY9UrTVWxAJvwOc5r0ig4PgovKMgKDIOxJIAojA2t55QRsAOwCtjz4Fc0EJqMspo0eEo93CM5fMtUmxiC4jOS2uugwZaequSDVTqmRKXlT2b4RslnrFwVemeQFHysUHPCsIAxyNKPCvvUieSnFowZjP9M7yVpnXk3ZIPgm40lVxftleSXjKtZCeinw_w7_jUSgT1m-k34UVikrH6Dft7F1SSBt7wGi65Dwijh8YNNLftdugRJiFMGBwDYxA",
    "expires_in": 299,
    "scope": "read openid profile write export",
    "token_type": "Bearer"
}


```

##### Using the authorization_code flow:

Here when you try to access a resource you'll need to log in first, obtain a code, use that code to get an access token.

In a real world situation, when you try to log in to a service you'll get redirected to the auth server to authenticate,
the URL will look like this:

```
http://localhost:8080/oauth2/authorize?response_type=code&client_id=client&scope=profile openid read write export&redirect_uri=http://localhost:4444
```

once authenticated, you are basically delegating your access to the app, the app then get a code, 
which will be used as proof to present to the auth server for token retrieval;

example of a code : ```UzgjnibbAWiVsyAo8b9YBcr-50hTSxbZAMqCgJt3Tj_r5xBeKyPpVknjI1FArTTCOaJVSq9AGsy4B3sIHUSIZOeCOxCk-W274biTqO5nOlRVZRxQPtsvUdUkQNgsNyoY```

request(x-www-form-urlencoded params):

````POST http://localhost:8080/oauth2/token?client_id=client&redirect_uri=http://localhost:4444&grant_type=authorization_code&code=jnVSWWnYUIwCIx2XFjlPniaU74qcfJJZuRYcFzyHFwmzlw9ElLmdjheu8zDFs--zybJ6T9KTvEM-66PIGp2yc-yn7soexm2oIO3OltmsV-5J7Q7PQS4mGKkDEGd3RBfX````

response: 

````json
{
    "access_token": "eyJraWQiOiJlNzQxYTJlNS03Njc3LTRkOGItYjk5Mi1lZGQyZTA4ZjgxMDMiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJ1c2VyIiwiYXVkIjoiY2xpZW50IiwibmJmIjoxNzE3OTQ3MjQ5LCJzY29wZSI6WyJyZWFkIiwib3BlbmlkIiwicHJvZmlsZSIsImV4cG9ydCIsIndyaXRlIl0sImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6ODA4MCIsImV4cCI6MTcxNzk0NzU0OSwiaWF0IjoxNzE3OTQ3MjQ5LCJqdGkiOiIxOTdjMjI4NS0zNTkxLTRmMjAtODg4MC0xOTRiMTVhOGM1OGUifQ.jBq-MBessWEoXiuO9A0fhl8fZwSni2bYeAQx_1XyP9Vezf_dAEtlSozMvyBiFT4dJMs75F3TW5Ffy5K3Njj5BErK1xMCcaL4OV3QNOHIaGAGGX_GMp1jhuhVkN8UOb4_tw_yxJN_QeHrVEIBXRKG-R61ff32yz0EZaGX0VvT619BBraI2QzieVyehYQ2ncentt6ONgSm8emzew3Y-WET6jLZzQmXUrm9qhgs5-u_uUlUTdW6vmsbQAvaQcfoofs0iyJptJAKoUx5t8IJVfFqAt2PUt9miC3MD23FWuJRzJLZmQUb-UgJj-oLfv6Rftb5yMZ92h89iDzDrXHzGoXfJw",
    "refresh_token": "4L9uUwDRDg2T4nHiehxa97WD7dIfWbMz93yRpmddkgfX37UUT2ilzPk7PignlVu3Wn5Fh8fu22Xqs2kdD4a86vJh6Uc_UDOdfU2IayqSPsrSt7rjOSVGo2_uwaYcDGAB",
    "scope": "read openid profile export write",
    "id_token": "eyJraWQiOiJlNzQxYTJlNS03Njc3LTRkOGItYjk5Mi1lZGQyZTA4ZjgxMDMiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJ1c2VyIiwiYXVkIjoiY2xpZW50IiwiYXpwIjoiY2xpZW50IiwiYXV0aF90aW1lIjoxNzE3OTQ3MjMxLCJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgwODAiLCJleHAiOjE3MTc5NDkwNDksImlhdCI6MTcxNzk0NzI0OSwianRpIjoiYTA1YTQyYzItNDk2NC00YWFmLThmNWUtMDM5M2QzNGJhMWNkIiwic2lkIjoia3NRaUx2SEN2TFZsb1Bfa0hPRHJGb0tWQ1p1aW5hTUhKdzY1ZEJ1eDlObyJ9.UmtKilvTonfqqlli5Y6JM1RmHX6jsE-QLaeHqFBoYofzGcshyBnvi4hNv1KeHiHXBpzhwGaKmOr6AqPPyHXaaY7fJPwKC68T7pyz5ZvTeNTXYUFK6By6wPasY5Ixoc3beOmkGvjb0zdtMa8mdFO2QIn7Wt-3lGf979AIOBROl6h0wvpyoj7YAU5IYgB9xnpgmgkRkmW0GBddQgi66VRI8rc11LPfs5gIG5XEPqPrXEeYoh-GbB_q2AUtNETHDRMuBSOfvLkIhaMLGOJ8soj9MwdOj2lFalBcW8AT4yFxJhPJHDpnZtlVa7MNzpnQo4S6xYCYjERZwT4XjGAuTjmfOQ",
    "token_type": "Bearer",
    "expires_in": 299
}
````

Your client need to have the authorization_code grant type configured at creation to be able to query the endpoint properly.