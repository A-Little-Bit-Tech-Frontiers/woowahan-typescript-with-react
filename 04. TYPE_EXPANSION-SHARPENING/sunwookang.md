# 타입 좁히기
- 변수 또는 표현식의 타입 범위를 더 작은 범위로 좁혀나가는 과정
- 더 정확하고 명시적인 타입 추론
- 타입 안정성 높임

## 타입 가드
- typeof
- instanceof
  - A instanceof B
  - A가 B 클래스에 속하는지 확인
- in
  - A in B
  - A 속성을 B 객체가 갖고있는지 확인
- is
  - 함수의 반환 타입 정의
  - ```
    interface Bird {
      fly: () => void;
    }
    
    interface Fish {
      swim: () => void;
    }
    
    // `is` 키워드를 사용한 사용자 정의 타입 가드
    function isFish(pet: Bird | Fish): pet is Fish {
      return (pet as Fish).swim !== undefined;
    }
    
    const pet: Bird | Fish = /* 어떤 로직을 통해 Bird 또는 Fish 타입의 객체를 얻는다 */;
    
    if (isFish(pet)) {
      pet.swim(); // `isFish` 타입 가드 덕분에, TypeScript는 이 블록 내에서 `pet`을 `Fish` 타입으로 간주합니다.
    } else {
      pet.fly(); // 여기서 TypeScript는 `pet`이 `Bird` 타입이라고 간주합니다.
    }
    ```


## 타입에 판별자 달기
- 태그 유니온, 판별 유니온, 구별된유니온 여러 이름으로 불림
- 여러 객체 타입을 유니온으로 활용하고자 할 때 유용

- 예시
```
type MaleSchool = 'male1' | 'male2' | 'male3' | 'male4';
type FemaleSchool = 'female1' | 'female2';
type CoedSchool = 'coed1' | 'coed2' | 'coed3';

interface BasePerson {
  name: string;
  age: number;
}

interface MalePerson extends BasePerson {
  gender: 'MALE';
  school: MaleSchool | CoedSchool;
}

interface FemalePerson extends BasePerson {
  gender: 'FEMALE';
  school: FemaleSchool | CoedSchool;
}

type Person = MalePerson | FemalePerson;

const person1: Person = {
  name: 'Jane',
  age: 25,
  gender: 'FEMALE',
  school: 'female1',
};

const person2: Person = {
  name: 'Jhon',
  age: 25,
  gender: 'MALE',
  school: '', // male1, male2, male3, male4, coed1, coed2, coed3 으로 강제
};

```


- orgnaization member invite에서 admin을 선택했을 경우 이름을 추가로 받아야 한다고 할 때

**기존**
```
export interface IInviteMemberForm {
  role: MemberRoleType;
  email: string;
}
```

**변경**
```
interface IBase {
  role: MemberRoleType;
  email: string;
}

// 추가 속성을 가질 수 있는 role과 해당 속성을 정의
interface IAdminSpecific {
  role: 'ADMIN';
  name: string; // ADMIN일 때만 필요한 속성
}

// ADMIN과 나머지 role을 조합한 IInviteMemberForm 타입 정의
export type IInviteMemberForm =
  | (IBase & IAdminSpecific)
  | (IBase & { role: Exclude<MemberRoleType, 'ADMIN'> });
```

- 조건에 따라 서로 다른 타입을 하나의 로직에서 필요할 때 사용하면 매우 유용할 듯 하다.

