# 전자 지갑 (e-Wallet)을 사용하여 Oracle Cloud DB에 연결

## SQL Developer or SQLPlus 사용

1. Git Repository에 업로드 된 전자 지갑 다운로드

2. 전자 지갑 압축 풀기:
    - 다운로드한 전자 지갑 파일 (*.zip)을 압축 해제합니다. 압축 해제하면 'tnsnames.ora', 'sqlnet.ora' 및 'cwallet.sso'와 같은 여러 파일이 포함되어 있습니다.

3. 환경 변수 설정:
    - 압축 해제한 디렉토리 경로를 시스템 환경 변수 'TNS_ADMIN'에 추가합니다. 이전 답변에서 설명한대로 이 과정을 수행하십시오.

4. Oracle Instant Client 설치:
    - Oracle Instant Client를 설치하지 않았다면 설치해야 합니다. 설치 후에는 'PATH' 환경 변수에 Oracle Instant Client의 경로를 추가합니다.

5. SQL*Plus 또는 SQL Developer로 연결:
    - 암호화된 전자 지갑을 사용하여 연결하려면, SQL*Plus 또는 SQL Developer와 같은 도구를 사용해야 합니다. VSCode의 Oracle Developer Tools 확장 프로그램은 전자 지갑을 사용하여 연결하는 것을 직접 지원하지 않습니다.

6. SQL*Plus를 사용한 연결:
    - 터미널 또는 명령 프롬프트에서 다음 명령을 실행합니다:
    
    ```
    bashCopy code
    sqlplus username/password@'wallet_db_alias'(DESCRIPTION=(ADDRESS=(PROTOCOL=TCPS)(HOST=myhostname)(PORT=myport))(CONNECT_DATA=(SERVICE_NAME=myservicename))(SECURITY=(SSL_SERVER_CERT_DN=mycertdn)))
    
    ```
    
    - 'username'과 'password'를 실제 사용자 이름과 암호로 변경하고, 'wallet_db_alias'를 tnsnames.ora 파일에 있는 별칭으로 변경합니다. 또한, 'myhostname', 'myport', 'myservicename', 'mycertdn'을 전자 지갑 파일의 실제 값으로 변경합니다.
7. SQL Developer를 사용한 연결:
    - SQL Developer를 열고, 새로운 연결을 설정합니다.
    - 연결 이름, 사용자 이름 및 비밀번호를 입력하고, 연결 유형을 'TNS'로 선택합니다.
    - 네트워크 별칭 드롭다운에서 tnsnames.ora 파일에 있는 별칭을 선택합니다.
    - 연결을 저장하고 테스트합니다.


## Oracle Developer Tools for VSCode

VSCode에서 Oracle Developer Tools for VSCode 확장 프로그램을 사용하여 TNS Alias를 사용해 Oracle Cloud DB에 연결하려면 다음 단계를 따르십시오:

1. 전자 지갑 다운로드
    - 앞서 설명한 대로 전자 지갑을 다운로드합니다. 'tnsnames.ora' 파일이 포함되어 있습니다.

2. 환경 변수 설정:
    - 압축 해제한 디렉토리 경로를 시스템 환경 변수 'TNS_ADMIN'에 추가합니다. 이전 답변에서 설명한대로 이 과정을 수행하십시오.

3. VSCode 연결 설정:
    - VSCode에서 'Explorer' 뷰를 열고, 'Oracle Developer Tools' 섹션을 찾습니다.
    - 'Connections' 아래에서 '+' 버튼을 클릭하여 새 연결을 추가합니다.
    - 연결 유형을 'TNS Alias'로 선택하고 다음 정보를 입력합니다:
        - TNS Alias: tnsnames.ora 파일에 있는 TNS Alias를 선택합니다.
        - User: 사용자 이름
        - Password: 비밀번호
    - 연결 정보를 입력한 후 'Test Connection' 버튼을 클릭하여 연결이 제대로 작동하는지 확인합니다.
    - 연결 테스트가 성공하면 'Save Connection'을 클릭하여 연결을 저장합니다.

4. 연결 사용:
    - 저장된 연결을 사용하여 데이터베이스에 쿼리를 실행하고, 테이블 및 기타 객체를 관리할 수 있습니다. 'Oracle Developer Tools' 섹션에서 저장된 연결을 클릭하여 사용할 수 있습니다.