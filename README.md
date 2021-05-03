# Documentation OAuth2 (SSO-ASN)

### Flow Grant Type Authorization Code
Grant type ini digunakan untuk aplikasi client yang bisa menyimpan nilai client secret. Contohnya adalah aplikasi server side (PHP, Java) atau aplikasi desktop/mobile yang bisa dicompile. Nilai client secret bisa kita simpan sebagai variabel yang tidak bisa dilihat umum.

* Buka browser, arahkan ke : 

  `http://localhost:8090/oauth/authorize?client_id=clientwebbased&response_type=code&redirect_uri=http://example.com`

* Kita akan diredirect ke url yang kita sebutkan pada variabel `redirect_uri` di langkah pertama di atas dengan ditambahi parameter authorization `code`. URL hasil redirectnya seperti ini:
  
  `http://example.com?code=COntoH`
  
* Lakukan request dari **aplikasi client** untuk menukar authorization `code` dengan `access_token` :
    
        POST: /oauth/token?grant_type=authorization_code&code=COntoH&redirect_uri=http://example.com
        Host: localhost:8090
        Content-Type: application/x-www-form-urlencoded
        Authorization: Basic Y29udG9oOmNvbnRvaA==

* Kita akan diberikan `access_token` dalam response JSON seperti ini :

        {
            "access_token":"08664d93-41e3-473c-b5d2-f2b30afe7053",
            "token_type":"bearer",
            "refresh_token":"436761f1-2f26-412b-ab0f-bbf2cd7459c4",
            "expires_in":43199,
            "scope":"write read"
        }


#
### Flow Grant Type User Password
Grant type ini biasanya digunakan bila pembuat aplikasi client sama dengan pembuat resource server. Sehingga aplikasi client diperbolehkan mengambil data username dan password langsung dari user. Contohnya: aplikasi Twitter android ingin mengakses daftar tweet untuk user tertentu. Walaupun demikian, penggunaan flow type ini tidak direkomendasikan lagi. Sebaiknya gunakan flow type _authorization code_ atau _client credentials_.

* Request token ke **Authorization server** :

        POST /oauth/token?client_id=mobileapp&grant_type=password&username=ismailpomalingo&password=rahasia 
        Host: localhost:8090
        Content-Type: application/x-www-form-urlencoded
        Authorization: Basic Y29udG9oOmNvbnRvaA==

* Kita akan diberikan access_token dalam response JSON seperti ini :

        {
            "access_token": "b714480d-6fc6-4f71-bdce-c3442e3ef897",
            "token_type": "bearer",
            "refresh_token": "145122d2-e620-4eb9-a420-56082493a27d",
            "expires_in": 43199,
            "scope": "read write admin"
        }
    
#
### Flow Grant Type Implicit
Grant type ini biasanya digunakan apabila aplikasi client tidak bisa menyimpan nilai client secret dengan aman. Contohnya adalah aplikasi JavaScript (misalnya: AngularJS, jQuery, Backbone.js, dsb) yang source codenya bisa dilihat umum.

* Buka browser, arahkan ke :

    `http://localhost:8090/oauth/authorize?client_id=clientspamobile&response_type=token`
    
* Kita akan diredirect ke url yang kita daftarkan pada **Resource server**. URL hasil redirectnya seperti ini:

    `http://example.com/#access_token=9b0d1167-1a25-436b-aff1-b4aa00af9f58&token_type=bearer&expires_in=43199&scope=read%20write`
  
* Untuk Keamanan, sebaiknya kita menambahkan nilai variabel `state` pada **_aplikasi client_**. Nilai variabel `state` ini disimpan sebagai session variable di sisi **_server aplikasi client_**. Kita akan gunakan nilai variabel `state` ini untuk verifikasi pada langkah selanjutnya. Contoh:

    `http://appclient.com/api/state/xyz123`
    
    Setelah dapat variabel `state` dari request di atas, gunakan untuk generate token
    
    `http://localhost:8090/oauth/authorize?client_id=clientspamobile&response_type=token&state=xyz123`
    
    Setelah sukses login, authorization server akan melakukan redirect ke url yang kita daftarkan, yaitu `http://localhost:10001/api/state/verify`. URL ini akan ditambahkan hash variable berisi token, sehingga isi lengkapnya seperti ini
    
    `http://appclient.com/api/state/verify#access_token=9b0d1167-1a25-436b-aff1-b4aa00af9f58&token_type=bearer&expires_in=43199&scope=read%20write`
    
#
### Flow Grant Type Client Credentials
Pada flow type ini, aplikasi client diberikan akses penuh terhadap resource yang diproteksi tanpa perlu meminta username dan password user. Biasanya digunakan bila aplikasi client dan aplikasi resource server dibuat oleh perusahaan yang sama.

* Request token ke **Authorization server** dengan memasang `client_id` dan `client_secret` pada header dengan cara **Basic Authentication**

        POST /oauth/token
        Host: localhost:8090
        Content-Type: application/x-www-form-urlencoded
        Authorization: Basic Y29udG9oOmNvbnRvaA==
        Body: grant_type=client_credentials

#
#### `refresh_token`
* Bila `access_token` expire, kita bisa meminta `refresh_token` dari **aplikasi client** sebagai berikut :

        POST /oauth/token?grant_type=refresh_token&refresh_token=436761f1-2f26-412b-ab0f-bbf2cd7459c4
        Host: localhost:8090
        Content-Type: application/x-www-form-urlencoded
        Authorization: Basic Y29udG9oOmNvbnRvaA==

* **Authorization server** akan memberikan respon `access_token` dan `refresh_token` yang baru dalam response JSON :

        {
            "access_token":"e425cee6-7167-4eea-91c3-2706d01dab7f",
            "token_type":"bearer",
            "refresh_token":"436761f1-2f26-412b-ab0f-bbf2cd7459c4",
            "expires_in":43199,"scope":"write read"
        }
        
#
#### `check_token`
* **Resource server** mengecek ke **authorization server** apakah token tersebut valid atau tidak dengan HTTP request seperti ini :

        POST /oauth/check_token?token=e425cee6-7167-4eea-91c3-2706d01dab7f
        Host: localhost:8090
        Content-Type: application/x-www-form-urlencoded
        Authorization: Basic Y29udG9oOmNvbnRvaA==

* **Authorization server** akan membalas request `check_token` dengan response seperti ini :

        {
            "active": true,
            "user_name": "ismailpomalingo",
            "scope": [ "write", "read" ],
            "authorities": ["root","administrator","admin","user"],
            "exp": 1619918783,
            "aud": [ "ssoasn" ],
            "client_id": "clientwebbased"
        }

#
Pranata Komputer [`PraKom`] Pemerintah Daerah Provinsi Gorontalo @2021
