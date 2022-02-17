# 문제 발생

[me.saro.commons.ftp](https://github.com/saro-lab/commons)의 recv 메소드를 이용해 원격지에있는 log파일을 로컬로 복사중 복사가 안되는 현상 발생  

# 소스 

* 예시
```java
import me.saro.commons.ftp.FTP;

public void main(...){
	String remoteFile = "/home/test/log/action.log";
	File localFile = new File("D:\dev\log\action.log");
	
	FTP ftp = null;
	try {
			ftp = FTP.openFTP(host, port, username, password);
		} catch (IOException e) {
			try {
				ftp = FTP.openFTPS(host, port, username, password);
			} catch (IOException e1) {
				try {
					ftp = FTP.openSFTP(host, port, username, password);
				} catch (IOException e2) {
					log.error("FTP 접속 실패");
				}
			}
		}
	
	String remoteFileName = Paths.get(targetPath).getFileName().toString();
	String remoteWorkingPath = remoteFile.replace(remoteFileName, '');
	ftp.cd(remoteWorkingPath);
	
	boolean isRun = ftp.recv(remoteFile, localFile);
	
	if(isRun){
		log.info("다운로드 완료");
	}else{
		log.error("다운로드 실패");
	}
	
}
```

# 원인

ftp.recv(~~remoteFile~~ -> **remoteFileName**, localFile);


# 원인 발생 확인

실제 recv 동작하는 소스를 확인  
![1](https://user-images.githubusercontent.com/48544100/154389647-04142d58-71b5-4362-b244-c5df45be5cf1.PNG)  
157번줄 원격지에 해당 파일이 존재하는 지 확인하는 메소드이다. 이 문제가 아닐까 생각하며 해당 구현된 메소드를 확인  
![2](https://user-images.githubusercontent.com/48544100/154389846-408f25a2-6a3f-4406-b2b9-24c7bfe11e0e.PNG)  
이상이 없어보이는 것 같은데 잘 생각해보니깐 나는 원격지의 **fileName만** 보내는게 아닌 **절대 경로 + 파일 명**을 보내고있었음.  
여기서 이상함을 느껴서 절대 경로와 파일명을 넘겨주는 것을 파일명만 보낼 수 있게 소스 변경 이후 테스트해보니 정상 동작.






