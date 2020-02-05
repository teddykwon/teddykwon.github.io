---
layout: post
title:  "Effective Java 2장"
tags: [study|공부]
---

스터디를 하면서 느꼈던 점들 중에서 이전에는 그냥 넘어갔던 부분인데 다시 보니 신경 쓰이는 부분이 있어서 보게 되었다.

# Builder 패턴
클래스 생성 시 setter로 값을 주입하기 전에 class가 완전 생성되지 않아 그 전에 사용이 이루어지는 등 위험성이 존재한다고 되어있어서 Builder 패턴을 쓰는게 좋고, 시간이 지나면 어차피 class는 늘어나기 때문에 좋다고 되어있다.

책에서 저렇게 병적으로 static을 써서 subclass를 추가해서 하는 것보다는 그냥 lombok @builder 정도만 사용해서 깔끔하게 유지하는게 좋을 것 같다.

# try-finally
이건 생전 처음보는 패턴이다. 얼마전까지 inputstream을 사용할 일이 있어서 사용했는데 finally 없이 안전하게 처리 할 수있다니.
close()를 호출하기 위해 무의미한 finally로 보다 자동으로 close되는 try-with-resources를 사용하자

# 다 쓴 객체 해제
이건 옛날에나 있었던 일이나 Excel관련 프로그램 만들 때, 너무 memory를 많이 써서 null로 변환해가며 작업했던 기억이 있다. 프로그램 실행 주기 내에 gc가 실행이 잘 안되거나 순환참조 형식으로 메모리 참조가 일어나서 해제가 안되는 등 일이 많았다. null로 변환해주는 것보다는 개발 할 때, block을 잘 생각해서 정리해야겠다.