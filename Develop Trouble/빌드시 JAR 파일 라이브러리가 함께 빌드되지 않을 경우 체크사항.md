프로젝트에서 동료 개발자 분이 라이브러리 추가를 하는 상황에 발생했던 트러블에 대한 기록이다.

프로젝트에서 필요한 라이브러리가 생겨 maven 을 통하여 추가하려 하였으나 maven 리포터지터리의 문제인지 제대로 받아지지 않았다. 따라서 해당 라이브러리 jar 을 직접 받아 프로젝트 내부에 추가하였다.

로컬에서 실행시 문제 없이 잘 동작하였으나 서버에 올리기 위해 빌드 후 배포 했을때 문제가 발생하였다. 해당 파일의 클래스를 찾을 수 없어 스프링 빈이 주입되지 않는다는 오류를 만나게 되었다. 

원인을 파악하기 위해 빌드 파일을 확인한 결과 해당 라이브러리 파일이 누락되어 빌드가 진행된다는 사실을 확인하게 되었다. 이것저것 시도하다 결국 해결하게 되었는데, 해결 방법의 정확한 이유는 파악하지 못하였다... 라이브러리 파일이름이 "라이브러리-1.0.1.jar" 형태였는데 뒷 부분의 버젼 부분을 변경하여 "라이브러리.jar"로 변경 한 이후에는 빌드시 제대로 포함되는 것을 확인하였다.

대충 추측을 하자면 "-1.0.1" 과 같이 하이픈(-)을 사용할 경우 빌드 시 파일이름 처리에 문제가 있는게 아닌가 싶다. "라이브러리.jar" 형태로 파일을 포함하여 빌드할 경우 최종 프로젝트 빌드파일에 추가된 이름이 "라이브러리-1.0.1.jar" 형태가 되는 것으로 보아 빌드과정에서 메이븐이 파일 이름을 변경하는 무언가가 있고, 미리 하이픈이 들어가 있을 경우 해당 파일 처리에 문제가 있는 것이 아닌가 싶다.

요약: 라이브러리 파일이름이 "라이브러리-1.0.1.jar"의 형태일 경우 하이픈을 포함한 버젼을 지워보자