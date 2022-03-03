### tomcat設定https

##### 首先你先要有憑證CA，而免費憑證機構有`Let's Encrypt`，可以提供免費生成
##### 而在git上已經有人寫好了腳本 `acme.sh`可以幫你生成該驗證，並且在每三個月過期後，可以設定排程自動重新申請。

> git位置:

    https://github.com/acmesh-official/acme.sh
    
> ***step 1.*** 下載 acme.sh [他是需要用curl 而不是常見的apt-get]
> 
    curl https://get.acme.sh | sh -s email=silvervane@gmail.com
    
>***step 2.*** 確認安裝 

    acme.sh -v
    
>***step 3.*** 現在acme.sh預設機構是ZeroSSL，需要更換至Let's Encrypt

    acme.sh --set-default-ca  --server  letsencrypt

### HTTP驗證 & DNS驗證    
##### HTTP驗證     
>***step 4.*** 
>>驗證機構會先給我們一個檔案，而我們需要把它放到伺服器上我們的網站根目錄，驗證機構會藉由網域去連上我們網站，如果連上並取得該檔案驗證成功，就證明這網站是我們所擁有!<br>
>> `-d 表示域名` <br>
>> `-w 表示你server所在網站根目錄，這裡是乾淨的tomcat所以只是預設根目錄所在ROOT`

    acme.sh --issue -d silvervane.ml -w /home/andy19910523/tomcat/ROOT/
    
>***step 5.***
>>當你***step4***生成後，預設在`/home/username/.acme.sh/silvervane.ml/`內生成你的key和cer檔
>>根據acme說明，這些檔案不要去移動，從以下指定去生成config所需要的pem檔

    acme.sh --install-cert -d silvervane.ml \
    --cert-file      /tomcat/coonfig/cert.pem  \
    --key-file       /tomcat/coonfig/key.pem  \
    --fullchain-file /tomcat/coonfig/fullchain.pem \
   
>***step 6.***
>>有了憑證就可以開始配置了，在`tomcat/conf/server.xml`補上
  
    <Connector port="8443" protocol="org.apache.coyote.http11.Http11AprProtocol"
      maxThreads="150" SSLEnabled="true" >
      <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
      <SSLHostConfig>
        <Certificate certificateKeyFile="conf/key.pem"
          certificateFile="conf/cert.pem"
          certificateChainFile="conf/fullchain.pem"
          type="RSA" />
      </SSLHostConfig>
   </Connector> 

>***step 7.***
>>但有以上配置，也還是不能成功導向https，你還需要在你專案底下web.xml配置以下
  
    <security-constraint>
       <web-resource-collection>
         <web-resource-name>test</web-resource-name>
         <url-pattern>/*</url-pattern>
         </web-resource-collection>
         <user-data-constraint>
           <transport-guarantee>CONFIDENTIAL</transport-guarantee>
         </user-data-constraint>
    </security-constraint>      
 >***step 8.***
 >>確認是否成功每次拜訪導向指定https的port
  
    curl http://silvervane.ml -I           //如果成功你可以看到有302導向指定https port
    
    ss -ntl                              //也可以看到8443有被監聽
 
 >***step 9.***
 >>最後 `http預設80`和`https預設443` ，那你就可以回頭指定port導向443，這樣在瀏覽器上方就不會出現port號囉!
