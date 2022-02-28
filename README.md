# Documentation OAuth2 (Account SSO)

### Flow Grant Type Authorization Code (SSO)
Grant type ini digunakan untuk aplikasi client yang bisa menyimpan nilai client secret. Contohnya adalah aplikasi server side (PHP, Java) atau aplikasi desktop/mobile yang bisa dicompile. Nilai client secret bisa kita simpan sebagai variabel yang tidak bisa dilihat umum.

* Buka browser, arahkan ke : 

  `https://accounts.gorontaloprov.go.id/oauth/authorize?client_id=example&response_type=code`

* Kita akan diredirect ke url yang kita sebutkan pada variabel `redirect_uri` di langkah pertama di atas dengan ditambahi parameter authorization `code`. URL hasil redirectnya seperti ini:
  
  `http://example.com?code=COntoH`
  
* Lakukan request dari **aplikasi client** untuk menukar authorization `code` dengan `access_token` :
    
        POST: /oauth/token?grant_type=authorization_code&code=COntoH
        Host: https://accounts.gorontaloprov.go.id
        Content-Type: application/x-www-form-urlencoded
        Authorization: Basic Y29udG9oOmNvbnRvaA==

* Kita akan diberikan `access_token` dalam response JSON seperti ini :

  ```json
  {
      "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
      "token_type": "bearer",
      "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expires_in": 43199,
      "scope": "trust read",
      "jti": "26ac654e-13dc-463a-abbb-2aec26149a91"
  }
  ```
  
#
### Flow Grant Type Implicit (Account SSO)
Grant type ini biasanya digunakan apabila aplikasi client tidak bisa menyimpan nilai client secret dengan aman. Contohnya adalah aplikasi JavaScript (misalnya: AngularJS, jQuery, Backbone.js, dsb) yang source codenya bisa dilihat umum.

* Buka browser, arahkan ke :

    `https://accounts.gorontaloprov.go.id/oauth/authorize?client_id=example&response_type=token`
    
* Kita akan diredirect ke url yang kita daftarkan pada **Resource server**. URL hasil redirectnya seperti ini:

    `http://example.com/#access_token=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...&token_type=bearer&expires_in=43199&scope=trust%20read%20write&jti=07107a28-a128-4c74-b2c7-cdedaaf90678`
  
* Untuk Keamanan, sebaiknya kita menambahkan nilai variabel `state` pada **_aplikasi client_**. Nilai variabel `state` ini disimpan sebagai session variable di sisi **_server aplikasi client_**. Kita akan gunakan nilai variabel `state` ini untuk verifikasi pada langkah selanjutnya. Contoh:

    `http://example.com/api/state/xyz123`
    
    Setelah dapat variabel `state` dari request di atas, gunakan untuk generate token
    
    `https://accounts.gorontaloprov.go.id/oauth/authorize?client_id=example&response_type=token&state=xyz123`
    
    Setelah sukses login, authorization server akan melakukan redirect ke url yang kita daftarkan, yaitu `http://example.com/api/state/verify`. URL ini akan ditambahkan hash variable berisi token, sehingga isi lengkapnya seperti ini
    
    `http://example.com/#access_token=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...&token_type=bearer&expires_in=43199&scope=trust%20read%20write&jti=07107a28-a128-4c74-b2c7-cdedaaf90678`

#
### Flow Grant Type User Password (Web Service)
Grant type ini biasanya digunakan bila pembuat aplikasi client sama dengan pembuat resource server. Sehingga aplikasi client diperbolehkan mengambil data username dan password langsung dari user. Contohnya: aplikasi Twitter android ingin mengakses daftar tweet untuk user tertentu. Walaupun demikian, penggunaan flow type ini tidak direkomendasikan lagi. Sebaiknya gunakan flow type _authorization code_ atau _client credentials_.

* Request token ke **Authorization server** :

        POST /oauth/token?client_id=mobileapp&grant_type=password&username=ismailpomalingo&password=rahasia 
        Host: https://accounts.gorontaloprov.go.id
        Content-Type: application/x-www-form-urlencoded
        Authorization: Basic Y29udG9oOmNvbnRvaA==

* Kita akan diberikan access_token dalam response JSON seperti ini :

  ```json
  {
      "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
      "token_type": "bearer",
      "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expires_in": 43199,
      "scope": "trust read",
      "jti": "26ac654e-13dc-463a-abbb-2aec26149a91"
  }
  ```
        
#
### Flow Grant Type Client Credentials (Web Service)
Pada flow type ini, aplikasi client diberikan akses penuh terhadap resource yang diproteksi tanpa perlu meminta username dan password user. Biasanya digunakan bila aplikasi client dan aplikasi resource server dibuat oleh perusahaan yang sama.

* Request token ke **Authorization server** dengan memasang `client_id` dan `client_secret` pada header dengan cara **Basic Authentication**

        POST /oauth/token
        Host: https://accounts.gorontaloprov.go.id
        Content-Type: application/x-www-form-urlencoded
        Authorization: Basic Y29udG9oOmNvbnRvaA==
        Body: grant_type=client_credentials

#
### `Refresh Token`
* Bila `access_token` expire, kita bisa meminta `refresh_token` dari **aplikasi client** sebagai berikut :

        POST /oauth/token?grant_type=refresh_token&refresh_token=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
        Host: https://accounts.gorontaloprov.go.id
        Content-Type: application/x-www-form-urlencoded
        Authorization: Basic Y29udG9oOmNvbnRvaA==

* **Authorization server** akan memberikan respon `access_token` dan `refresh_token` yang baru dalam response JSON :

  ```json
  {
      "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
      "token_type": "bearer",
      "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expires_in": 43199,
      "scope": "trust read",
      "jti": "26ac654e-13dc-463a-abbb-2aec26149a91"
  }
  ```
        
#
### `Check Token`
* **Resource server** mengecek ke **authorization server** apakah token tersebut valid atau tidak dengan HTTP request seperti ini :

        POST /oauth/check_token?token=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
        Host: https://accounts.gorontaloprov.go.id
        Content-Type: application/x-www-form-urlencoded
        Authorization: Basic Y29udG9oOmNvbnRvaA==

* **Authorization server** akan membalas request `check_token` dengan response seperti ini :

  ```json   
  {
      "aud": [ "sso"],
      "user_name": "ismailpomalingo",
      "scope": [ "trust", "read" ],
      "active": true,
      "exp": 1646107545,
      "authorities": ["root","administrator","admin","user"],
      "jti": "26ac654e-13dc-463a-abbb-2aec26149a91",
      "client_id": "example"
  }
  ```

#
### `Longout`

* Buka browser, arahkan ke : 

    `https://accounts.gorontaloprov.go.id/oauth/logout?token=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...&redirect=http://example.com`
    
* **Authorization server** akan menjalankan Logout secara Global pada semua **Resource server** yang menggunakan layanan Service **Account SSO**.

#
### OpenID Connect (OIDC)
OpenID Connect (OIDC) mendefinisikan alur masuk yang memungkinkan aplikasi klien (third party) untuk mengautentikasi pengguna, dan untuk mendapatkan informasi (atau "klaim") tentang pengguna tersebut, seperti user_name, scope, dan sebagainya.

#### `PUBLIC KEY`
* Buka browser, arahkan ke : 
    `https://account.gorontaloprov.go.id/oauth/token_key`
  
        {
        alg: "SHA256withRSA",
        value: "-----BEGIN PUBLIC KEY----- MIIBIjANBgkqhkiG9w0BAQEFAAO.. -----END PUBLIC KEY-----",
        }
        
#### `JSON Web Key`
* Buka browser, arahkan ke : 
    `https://account.gorontaloprov.go.id/oauth/.well-known/jwks.json`
  
        {
          keys: [
            {
              kty: "RSA",
              e: "AQAB",
              n: "lEdwA7sdAmq5LXicUE2v2raTteoVIj6FcO..",
            }
          ]
        }
        
#
Pranata Komputer [`PraKom`] Pemerintah Daerah Provinsi Gorontalo @2021
