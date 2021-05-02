---
title:  "Cloud Computing"
categories: programming
tags: Infrastructure
---

지금까지 나는 개인 프로젝트 할 때 1. ***무료여서*** 2. ***집에 컴퓨터를 계속 켜놓을 수 없어서*** AWS를 사용하고, '클라우드 컴퓨팅'을 사용했다고 표현했다. 최근에 이 표현이 맞는지, 실제 업무에서는 어떤 이유료 사용하고 있는지 알아볼 필요성을 느껴 정리해보게 되었다.

## On Premises vs Cloud Computing

먼저 On Premises가 뭔지 알 필요가 있다. On Premises는 회사가 직접 데이터 센터를 설치하여 사용할 수 있도록 하는 것이다. 따라서, 해당 회사의 개발자는 물리적으로 서버에 접근이 가능해지고 인프라 및 데이터의 구성, 관리 및 보안을 직접 제어할 수 있다. 그리고 Cloud Computing은 회사가 데이터 센터를 직접 관리하지 않고 인터넷을 통해 컴퓨팅 리소스를 추상화하여 사용하는 것이다.

On promises와 Cloud Computing은 다음과 같은 차이가 있다. 따라서 회사는 요구 사항과 솔루션에 따라 적절한 방법을 사용해야 된다.

### 위치

- On Premises: 소프트웨어가 작동하는 데 필요한 하드웨어 및 기타 인프라 등이 회사의 내부 시스탬에서 설정된다.
- Cloud Computing: 일반적으로 인터넷을 통해 접근이 가능하여 시간과 위치의 제약이 없다. 접근하기 위해서 애플리케이션이나 웹 브라우저만 설치하면 된다.
  
### 서비스 접근에 필요한 비용

- On Premises: 하드웨어, 기타 인프라 구매, 소프트웨어 설치 및 검사 등 초기 비용이 매우 높다. 또한 지속적인 유지 보수 및 운영에 비용이 든다.
- Cloud Computing: 사용자는 접근 할 수 있는 디바이스만 있으면 되기때문에, 초기 비용이 거의 들지 않는다. 하지만 정기적인 구독료가 발생하므로 장기적인 비용에는 Cloud Computing의 비용이 적게 든다고 보장할 수 없다.

### 백업 및 데이터 저장

- On Premises: 회사에서 소프트웨어 데이터의 백업 및 저장을 직접 관리해야된다. 즉, 데이터와 보안에대한 완전한 통제권을 가질 수 있다.
- Cloud Computing: Cloud Computing 제공 업체에서 데이터 백업을 하게된다. 하지만 회사에서 해당 데이터와 보안을 통제할 수 없을 수 있다. 따라서, 금융 기관과 같은 데이터 보안에 대해 높은 수준의 규제를 적용받는 경우 문제가 발생할 수 있다.

### 데이터 보안

- On Premises: 회사에서 데이터 관리의 완전한 통제권을 가질 수 있다.
- Cloud Computing: 데이터 관리의 완전한 통제권을 가질 수 없을 수도 있다. 하지만, 일반적인 기업보다 데이터 보안 시스템에 많은 투자를 하고 있어 더 높은 보안 기술을 가지고 있다는 장점도 있다.

최근에는 1. ***데이터를 어떻게 관리할 것인가*** 2. ***서버를 어떻게 관리할 것인가*** 3. ***네트워크를 어떻게 관리할 것인가***에 대한 것은 서비스 제공자가 관심에서 분리하는 것을 선호하고 있기 때문에 Cloud Computing을 많이 사용하고 있는 추세다.

## Public Cloud vs Private Cloud

클라우드의 소유권에 따라 여러 가지로 분류가 가능하다.

- Public Cloud: 공용 인터넷을 통해 제공되는 클라우드다. 우리가 흔하게 생각하는 클라우드가 이에 해당한다. 대표적인 예로 AWS, Google Cloud, IBM Cloud, Microsoft Azure 등이 있다.
- Private Cloud: 단일 사용자 또는 그룹 전용 클라우드 환경으로, 사용 시에 단일 사용자 또는 그룹의 방화벽으로 보호된다. 완전히 독립적인 접근 권한이 있는 사용자에게만 클라우드를 제공하는 형태의 클라우드를 말한다.

## IaaS vs PaaS vs SaaS

클라우드 서비스 제공업체가 어디 까지를 제공하냐에 따라서도 분류가 된다. 이 내용은 글보다 그림을 통해 어디까지 제공하는지 보는 것이 명확하게 설명 가능하다.

![iaas-paas-saas](/assets/images/iaas-paas-saas.png)

- IaaS: 종량제 서비스로, 필요한 경우 제3사가 스토리지와 가상화 같은 인프라 서비스를 인터넷을 통해 제공한다.
  - 예시: AWS, Microsoft Azure, Google Cloud
- PaaS: 하드웨어와 소프트웨어를 호스팅하고 이러한 플랫폼을 사용자에게 통합 솔루션, 솔루션 스택 또는 인터넷을 통한 서비스롤 제공한다
  - 예시: AWS Elastic Beanstal, Heroku, Red Hat OpenShift
- SaaS: 모든 애플리케이션은 제공업체가 관리하며 웹 브라우저를 통해 제공한다.
  - 예시: Dropbox, Salesforce, Google Apps

## 참고 자료

<https://www.webopedia.com/definitions/on-premises/>

<https://en.wikipedia.org/wiki/On-premises_software#Location>

<https://www.redhat.com/ko/topics/cloud-computing/public-cloud-vs-private-cloud-and-hybrid-cloud>

<https://www.redhat.com/ko/topics/cloud-computing/iaas-vs-paas-vs-saas>