# ทำการ JWT Authentication APIs ด้วย Keycloak

ตัวอย่างการทำใช้ Dotnet 6 Minimal สร้าง Web API เพื่อทำการ Authentication กับ Keycloak 
มีประโยชน์ในการทำ Single Sign On (SSO) ระหว่าง Frontend กับ Backend หรือระหว่าง Backend ด้วยกัน
ทำให้แยก Microservice ได้ง่าย สร้าง realm (ตัวอย่างชื่อ frappet) ใน Realm Setting เก็บค่า RS256 kid ไว้ เป็น Public key ที่จะใช้ Verify token ใน appsetting.json

 ![Realm Setting Keys](./realm-setting.png)

 ## สร้าง API เซิร์ฟเวอร์ ด้วย .NET
เมื่อ Authentication กับ Realms นี้ จะได้ Access Token 
สร้าง dotnet 6 minimal apiใช้โค้ดและเก็บ Public Key ใน appsetting.json ตามโค้ดด้านล่าง แล้วรันบน localhost ก็ได้ มันจะรองรับ Token ของ Realms นี้

### คำสั่ง

    dotnet --version
    6.0.202
    dotnet new webapi -minimal -o dotnet6-demo
    cd .\dotnet6-demo\
    dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer  
    dotnet restore
    code .

### ไฟล์ appsetting.json

    {
    "Jwt": {    
        "Key": "UHLdOu3xxxxxxxxxxWRgu220joK3-acaxxxxxUHC6k",    
        "Issuer": "https://keycloak.xxx.com/auth/realms/frappet"    
    },      
    "Logging": {
        "LogLevel": {
        "Default": "Information",
        "Microsoft.AspNetCore": "Warning"
        }
    },
    "AllowedHosts": "*"
    }

### ไฟล์ Proram.cs

    using Microsoft.AspNetCore.Authentication.JwtBearer;
    using Microsoft.IdentityModel.Tokens;
    using System.Text;
    var builder = WebApplication.CreateBuilder(args);
    var issuer = builder.Configuration["Jwt:Issuer"];
    builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme).AddJwtBearer(opt =>
    {
        opt.RequireHttpsMetadata = true;
        opt.Authority = issuer;
        opt.TokenValidationParameters = new()
        {
            ValidateIssuer = true,
            ValidateAudience = false,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = issuer,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
        };
    });
    builder.Services.AddAuthorization();
    var app = builder.Build();
    app.UseAuthentication();
    app.UseAuthorization();

    app.MapGet("/", () => "Hello World!").RequireAuthorization();
    app.Run();

## แบบ 1 Authentication ด้วย secret  

จะมี secret ที่รู้กันระหว่าง client ของ keycloak และ ผู้ที่จะใช้ API  
Client ทำ Authentication กับ Keycloak ด้วย secret จะได้ Access key กลับมา ให้เอาไปแนบตอนเรียก API
วิธีนี้เหมาะกับการคุยกันระหว่าง Microservice ตัวอย่างนี้เอามาจาก  

<https://www.mshowto.org/keycloak-server-ile-apilariniza-jwt-validasyonu-nasil-ekleyebilirsiniz-bolum-1.html>  

<https://www.mshowto.org/keycloak-server-ile-apilariniza-jwt-validasyonu-nasil-ekleyebilirsiniz-bolum-2.html>

### วิธีตั้งค่า ตั้งค่า client ของ keycloak


- สร้าง client ชื่อ dotnet6-demo โดยมี root url เป็น https://keycloak.xxx.com 
- เซ็ตค่าดังนี้  
Access Type : confidential  
Service Account Enabled : On  
Access Token Lifespan : 20  
Browser Flow: borwser
Direct Grant Flow : direct grant 

![Client Setting](./client-setting.png)

- ในแทป Credentials  ก็อปค่า Secret เก็บไว้ (ถ้ายังไม่มี หรือต้องการเปลี่ยนให้กดปุ่มเพื่อสร้างใหม่)
![Client Secret](./client-secret.png)

### ทดสอบ Authentication ด้วย Postman

- POST โดยแนบ Sercret ตามภาพจะได้ access_token

![Get Token](./access-token.png)

- เอา Access Token ไปใช้เรียกใช้ API ของ .NET

![Use Token](./api-authen.png)

## กรณี 2 Authentication ด้วย User/Password 

กรณีนี้เหมาะกับการใช้ JavaScript Framework(Vue,React,Svelte) สร้างฟอร์มรับ ชื่อยูสซอร์ และรหัสผ่าน เพื่อ authen กับ keycloak   


- กรณีใช้ keycloak-js ในการทำเพราะจะจัดการ refresh token, cookie ให้ด้วย Realm frappet สร้าง keycloak client ชื่อ js-console 
![Client Secret](https://github.com/schooltechx/youtube/raw/main/keycloak/sso-google/js-console-setting.png)
- สำหรับการ development สามาถตั้ง Valid Redirect URL เป็น http://localhost:3000/* ได้


## Frontend (มี User loin)
- ตั้ง webserver ด้วย nginx ใช้โค้ดตัวอย่าง [js-console](https://github.com/keycloak/keycloak/tree/main/examples/js-console/src/main/webapp) ไปใส่ไว้ใน wwwroot

- แก้ keycloak.json ให้ถูกต้อง

        {
        "realm" : "frappet",
        "auth-server-url" : "https://keycloak.xxx.com/auth",
        "ssl-required" : "external",
        "resource" : "js-console",
        "public-client" : true
        }

- เข้าหน้า https://keycloak-js-console.xxx.com/ จะทำการ redirect ไปหน้า login

- เอา Token ที่ได้หลัง login มาใส่ใน Postman เพื่อเรียก API ได้เหมือนกรณี 1
![Token from js-console](./js-console-show-detail.png)

JavaScript Frontend ควรใช้ [keycloak-js](https://github.com/keycloak/keycloak-documentation/blob/main/securing_apps/topics/oidc/javascript-adapter.adoc) จัดการจะทำให้ง่ายและปลอดภัยกว่า ดูโค้ดของ [js-console](https://github.com/keycloak/keycloak/tree/main/examples/js-console/src/main/webapp) เป็นตัวอย่างได้

- ตัวอย่างการ curl เพื่อขอ access token โดยไม่ได้ใช้ login popup ของ keycloak
``` sh 
curl -X POST 'http://keycloak.xxx:com/auth/realms/frappet/protocol/openid-connect/token' \
 --header 'Content-Type: application/x-www-form-urlencoded' \
 --data-urlencode 'grant_type=password' \
 --data-urlencode 'client_id=js-console' \
 --data-urlencode 'client_secret=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx' \
 --data-urlencode 'username=employee1' \
 --data-urlencode 'password=mypassword'
```
## Note

### ข้อมูลของ Realm

จะอยู่ในลิงค์นี้
<https://keycloak.xxx.com/auth/realms/frappet/.well-known/openid-configuration>

### Endpoint

- /protocol/openid-connect/auth  
- /protocol/openid-connect/token  
- /protocol/openid-connect/userinfo

### Mobile

กรณี mobile มันจะ redirect กลับมาที่ App  ยังไม่ได้ทดสอบ แต่น่าจะทำเหมือนกรณี 2 
มีตัวอย่างของ Flutter ที่ยังไม่ได้ลองเหมือนจะใช้ Flow คนละแบบ  
<https://iabhishek3.medium.com/keycloak-integration-flutter-7af4088ff7a>

### อ่านเพิ่มเติม

- [Vue with keycloak](https://medium.com/keycloak/secure-vue-js-app-with-keycloak-94814181e344)
- [การใช้งาน Keycloak เบื้องต้น](https://dev.to/mossnana/keycloak-2kl6)  
- [Integrating OpenID Connect to your application stack (Asp.Net Core, Angular)](https://dev.to/kayesislam/integrating-openid-connect-to-your-application-stack-25ch)
- [ASP.Net Core & Angular OpenID Connect using Keycloak](https://medium.com/@xavier.hahn/asp-net-core-angular-openid-connect-using-keycloak-6437948c008)
- [Keycloak and Express](https://medium.com/keycloak/keycloak-and-express-7c71693d507a)

- [ตัวอย่างของ node.js](https://medium.com/devops-dudes/securing-node-js-express-rest-apis-with-keycloak-a4946083be51)