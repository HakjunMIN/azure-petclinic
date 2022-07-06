# Helm 라이브러리 차트 가이드

라이브러리 차트를 이용하여 손쉽게 Helm Chart를 생성할 수 있음. 특히 잘 만들어진 라이브러리 차트를 이용하면 value정의만으로도 이용할 수 있음.

## 직접 라이브러리 차트 생성

라이브러리 차트를 직접 생성하여 프로젝트 내 표준 차트로 사용할 수 있음.
가이드: https://helm.sh/docs/topics/library_charts/ 

> [!IMPORTANT]
> 대안으로 아래 오픈소스 라이브러리 1)차트를 직접 이용하거나 2) 클론하여 직접 template을 변경하여 helm repo (ACR사용가능)에 배포하여 프로젝트에서 이용하게 하거나 3) 이 차트를 이용하고 일부 template만 override (template폴더에 필요한 부분만 수정하여 재정의)하여 사용할 수 있음.
## 오픈소스 라이브러리 차트 사용

이 리파지토리의 Helm Chart는 라이브러리 차트로 리팩토링 됨.

1. 사용 라이브러리 차트: https://github.com/k8s-at-home/library-charts/tree/main/charts/stable/common

    > [!Note]
    > https://docs.k8s-at-home.com/our-helm-charts/development/creating-a-new-chart/를 사용하여 생성할 수도 있음.

2. `Chart.yaml` 디펜던시 정의

    ```yaml
    dependencies:
    - name: common
        repository: https://library-charts.k8s-at-home.com
        version: 4.4.2
    ```

3. templates디렉토리 기존 내용 지우고 `common.yaml`과 `NOTES.txt` 만 생성

    `common.yaml`:

    ```yaml
    {{ include "common.all" . }}
    ```

4. `values.yaml`에서 필요한 것들만 아래 전체 항목이 정의된 문서에서 Pick하여 정의

    전체 value값 정의: https://github.com/k8s-at-home/library-charts/tree/main/charts/stable/common#values

    본 리파지토리에서는 service와 deployment 정도 사용하므로 아래와 같이 `values.yaml`정의

    ```yaml

    replicas: 2
    image:
    repository: azurespringacr.azurecr.io/azurespring
    tag: 2.0-RC1
    pullPolicy: Always

    imagePullSecrets: 
    - name: acr-secret

    service:
    main:
        type: LoadBalancer
        ports: 
        http:
            port: 8080
            targetPort: 8080

    ingress:
    main:
        enabled: false
        annotations:
        kubernetes.io/ingress.class: addon-http-application-routing
        path: /
        hostname: VALUE_TO_BE_OVERRIDDEN

    env:  
    - name: APPINSIGHTS_INSTRUMENTATIONKEY
        value: VALUE_TO_BE_OVERRIDDEN
    ```

5. 환경 별 value파일을 만들어서 사용 

    values-qa.yaml
    ```yaml
    service:
    main:
        ports: 
        http:
            targetPort: 80
    ```

6. value정보들은 뒤에 올수록 우선순위가 높게 override되므로 아래와 같이 적용할 수 있음.

    ```bash
    $ helm upgrade --install sampleapp -f values-qa.yaml
    ```
    
    sampleapp의 기본값인 values.yaml에 values-qa.yaml에서 정의한 값만 override되어 생성됨

    > [!NOTE]
    > 아래와 같이 helm명령어의 set value로도 오버라이드할 수 있음.

    ```bash
    $ helm upgrade --install sampleapp --set imagePullSecrets[0].name=azurespringacr6471fd99-auth
    ```

6. 작성이 완료되면 library chart를 다운받기 위해 아래명령어 수행

    ```bash
    $ helm dependency update
    ```

7. 테스트

    `template`명령어로 Kubernetes Manifest를 생성 테스트

    ```bash
    $ helm template testapp ./sampleapp
    ```
