## Chapter 4

### Rounting

- Next.js의 라우팅은 **파일 시스템** 을 이용하여 구성된다.
- 중첩된 경로를 만드려면, 폴더를 중첩하고 내부에 `page.tsx` 파일을 추가하면 된다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/8902201d-7403-4f8e-92ac-80a62d62f226/9322aaee-81ca-45d8-89c4-7305ca110647/Untitled.png)

### Layout

- `layout.tsx` 를 이용하여 여러 페이지 간 공유되는 UI를 만들 수 있다. ex) 페이지 네비게이션
- `layout.tsx` 컴포넌트는 `children` `prop`을 받게 되는데, 이 `prop`은 컴포넌트 or 또다른 레이아웃
- 이러한 레이아웃은 하위 라우팅 페이지들에 자동으로 상속된다.
- 레이아웃을 사용하면 탐색 시 페이지 컴포넌트만 업데이트 되고, 레이아웃은 리렌더링이 일어나지 않는다. 이를 부분 렌더링이라고 하며, 성능상 이점을 가질 수 있다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/8902201d-7403-4f8e-92ac-80a62d62f226/f10d059e-b590-4ecc-a8b3-6f4198ea7182/Untitled.png)

## Chapter 5

### Link Component

- 페이지 간 링크를 만들어 내는 전통적인 방법은 `<a>` `HTML` 태그이지만, 이를 이용하면 페이지 전체가 새로고침되는 문제가 발생한다.
- `Next.js` 에서 제공하는 `Link` 컴포넌트를 이용하면, 페이지 새로고침 없이 페이지 탐색이 가능하다.
- 분명히 어플리케이션은 서버에서 렌더링되지만, 새로 고침이 없어 기존의 `SPA` 앱처럼 느껴진다. 그 이유는 `<Link>` 컴포넌트가 뷰포트에 표시될 때 마다, `Next.js` 링크된 경로의 코드를 백그라운드에서 `prefetches` 하기 때문이다. 사용자가 링크를 클릭할 때 쯤이면 이미 백그라운드에서 로드가 완료된 상태이므로 페이지 전환이 즉각적으로 이루어지는 것!

### usePathname Hook

- `Next.js` 에서 제공하는 `usePathname` 훅을 이용하면 현재 위치한 경로를 불러올 수 있다.
- 이는 현재 위치한 곳을 하이라이팅해주어야 하는 네비게이션 바와 같은 UI에 유용하다.
- 컴포넌트 상단에 `'use client';` 를 작성하여 클라이언트 컴포넌트로 바꿔주어야 함에 유의

## Chapter 6

### Set database

- `Vercel` 에서 제공하는 데이터베이스 서비스를 이용하면 편하게 데이터베이스를 프로젝트에 연동할 수 있다.
- 페이지 내에서 데이터베이스를 탐색하고 쿼리문을 동작시키는 기능도 제공

## Chapter 7

### Fetch data

- 기본적으로 `Next.js` 는 `**React` 서버 컴포넌트**를 통해 데이터를 가져온다. 이는 여러 가지 특징을 지니는데,
- `useState`, `useEffect`, 혹은 다른 데이터를 가져오는 라이브러리 없이 `async/await` 구문을 사용할 수 있다.
- 서버 컴포넌트는 서버에서 실행되므로 비용이 많이 드는 데이터 가져오기와 로직은 서버에 남겨두고 결과만 클라이언트로 전송할 수 있다.
- 또한, 서버에서 실행되므로 추가 API 계층 없이 데이터베이스를 직접 쿼리문을 실행할 수 있다.

### Request waterfall

```tsx
const revenue = await fetchRevenue();
const latestInvoices = await fetchLatestInvoices(); // wait for fetchRevenue() to finish
const {
  numberOfInvoices,
  numberOfCustomers,
  totalPaidInvoices,
  totalPendingInvoices,
} = await fetchCardData(); // wait for fetchLatestInvoices() to finish
```

- `Waterfall` 은 이전 요청의 완료 여부에 따라 달라지는 일련의 네트워크 요청들을 말한다.
- 위 코드의 경우 각각의 요청은 이전 요청이 데이터를 반환한 후에 본인의 요청을 시작한다.
- 이러한 패턴이 유용할 경우도 분명히 있다. 예를 들면 아이디를 확인하고, 확인이 됐다면 개인정보를 불러오고, 잘 불러왔다면 친구 목록을 불러오고..
- 하지만 이런 코드는 의도치 않게 성능에 영향을 끼칠 수 있다. 각 요청이 얼마나 걸릴 지 확실하지 않다면 더더욱!

```tsx
const data = await Promise.all([
      invoiceCountPromise,
      customerCountPromise,
      invoiceStatusPromise,
    ]);
```

- 위 코드처럼 `Promise.all()` , `Promise.allSettled()` 를 통해 모든 `Promise` 를 동시에 실행하여 병렬적으로 요청을 처리할 수 있다.
- 이렇게 `JavaScript` 네이티브 패턴에 의존한다면, 한 가지 단점이 있다. 하나의 데이터 요청이 유달리 다른 모든 데이터 요청보다 느리다면 어떨까?

## Chapter 8

### `Static Rendering`  이란?

- `Static Rendering`(정적 렌더링)을 사용하면 프로젝트 빌드(배포) 시 or 재검증 시 서버에서 데이터를 가져오기와 렌더링이 이루어진다. 그 후 결과는 컨텐츠 전송 네트워크(CDN)에 배포 및 캐시될 수 있다.
- 이는 사용자가 어플리케이션을 방문할 때 마다 캐시된 결과를 제공할 수 있다는 의미
- 빠르게 웹사이트에 접근이 가능해지고, 서버 부하가 줄어들며, SEO 성능 향상에 이점이 있다.
- 정적 블로그 게시물이나 제품 페이지와 같이 데이터가 없거나 사용자간 공유되는 데이터가 없는 UI의 경우 유용하게 사용할 수 있다.
- 반면 정기적으로 업데이트되는, 개인화된 데이터가 있는 대시보드에는 적합하지 않을 수 있다.

### `Dynamic Rendering` 이란?

- `Dynamic Rendering`(동적 렌더링)을 사용하면 사용자가 페이지를 방문(요청) 시 각 사용자에 대한 컨텐츠가 서버에서 렌더링된다. 동적 렌더링을 이용하면 아래와 같은 장점이 있다.
- 어플리케이션에서 실시간 또는 자주 업데이트되는 데이터를 표시할 수 있다. 데이터가 자주 변경되는 UI에 이상적
- 대시보드나 사용자 프로필과 같은 개인화된 콘텐츠를 제공하고 사용자 상호 작용에 따라 데이터를 업데이트하는 것이 더 쉽다.
- 쿠키나 URL 검색 매개변수와 같이 요청 시점에만 알 수 있는 정보에 액세스할 수 있다.
- 하지만, 지연이 심한 데이터 요청이 있을 경우, 페이지가 로딩되는 데 긴 시간이 걸릴 수 있다.
- 즉, 동적 렌더링을 사용하면 **페이지가 가장 느린 데이터 패칭만큼만 빨라질 수 있다.**

## Chapter 9

### Streaming

- `Streaming` 은 경로를 작은 `Chunks` 들로 나누고, 준비되는 대로 서버에서 클라이언트로 점진적으로 스트리밍할 수 있는 데이터 전송 기술이다.
- 이를 이용하면 느린 데이터 요청으로 전체 페이지가 차단되는 것을 방지할 수 있다.
- `React` 의 각 컴포넌트를 하나의 `Chunk` 로 간주할 수 있기에 컴포넌트 모델과 잘 어울린다.
- `Streaming` 을 구현하는 방법은 `loading.tsx` 를 이용하는 방법과 `<Suspense>` 를 이용하는 두 가지 방법이 있다.

### Route group 활용

- `()` 로 감싼 폴더는 URL 상의 경로에 포함되지 않는다.
- 따라서 `/dashboard/(overview)/page.tsx` 는 `/dashboard` 가 된다.
- `Route group` 을 이용하여 스켈레톤이 자식 컴포넌트에 전파되는 것을 막을 수 있다.

### Suspense

- `Suspense` 를 이용하면 동적 컴포넌트를 래핑하여 동적 컴포넌트가 로드되는 동안 인자로 전달한 `fallback` 컴포넌트를 표시할 수 있게 해준다.
- 묶어서 렌더링 할 필요가 있는 컴포넌트(ex. `Card`)는 래핑하여 `Suspense` 안에 넣어서 처리하자.
- `Suspense` 의 경계를 어디 두느냐에 따라 다양한 활용이 가능하다. 잘 사용해보자.

## Chapter 10

### Partial prerendering

- `Next.js 14` 에서 추가된 실험적 기능으로, 쉽게 설명해 정적으로 `route`를 렌더링하되,  컴포넌트 요소 중 동적으로 렌더링해야 하는 부분은 부분적으로 동적으로 렌더링하는 방법
- 초기 로딩과 전체 로딩 시간이 빨라진다는 장점이 있다고 하는데.. 일단 현재로서는 실험적인 기능이라고 한다.

## Chapter 11

### URL seach params

- `URL` 매개변수를 통해 검색하면 여러 이점이 존재한다.
- `URL` 을 사용자가 북마크하거나, 다른 사람에게 그대로 공유할 수 있다.
- `URL` 매개변수를 서버에서 직접 사용하여 초기 상태를 렌더링할 수 있으므로 서버 렌더링을 더 쉽게 처리할 수 있다.
- 검색 쿼리와 필터를 `URL`에 직접 넣으면 추가적인 클라이언트 측 로직 없이도 사용자 행동을 더 쉽게 추적할 수 있다.

### 검색 기능을 구현하는 데에 사용되는 Hook들

useSearchParams 

- 현재 `URL` 의 매개변수에 액세스할 수 있다.
- `URL` `/dashboard/invoices?page=1&query=pending` 의 검색 매개변수 = `{page: '1', query: 'pending'}`

usePathname

- 현재 `URL`의 경로명을 읽을 수 있다.
- `/dashboard/invoices` 의 경로명 = `'/dashboard/invoices'`

useRouter

- 클라이언트 컴포넌트 내에서 프로그래밍 방식으로 경로를 탐색할 수 있다.

## Chapter 12

### React Server action

- `Server action` 을 사용하면 서버에서 직접 비동기 코드를 실행 가능
- 클라이언트에서 발생하는 요청이 서버 컴포넌트를 통해 서버로 전달되고, 서버에서는 이 요청에 대한 처리를 수행하는 방식으로 동작한다.
- 클라이언트-서버 간 통신을 최적화하고, 보안과 성능을 개선하는 데 큰 도움을 준다.

```tsx
// Server Component
export default function Page() {
  // Action
  async function create(formData: FormData) {
    'use server';
 
    // Logic to mutate data...
  }
 
  // Invoke the action using the "action" attribute
  return <form action={create}>...</form>;
}
```

- `Server action`  를 이용하는 `form` 태그 예시, `action` 은 자동으로 `FormData` 객체를 받는다.
- 서버 액션은 Next.js 캐싱과도 긴밀하게 통합되어 있습니다. 서버 액션을 통해 양식이 제출되면 해당 액션을 사용하여 데이터를 변경할 수 있을 뿐만 아니라 revalidatePath 및 revalidateTag와 같은 API를 사용하여 관련 캐시의 유효성을 다시 검사할 수도 있습니다.

## Chapter 13

### error.tsx

- 예상치 못한 에러들을 `catch-all` 할 수 있는 방법.
- 에러 핸들링을 원하는 경로에 `error.tsx` 파일을 추가하고, `error` 데이터를 전달받아 핸들링하고 원하는 컴포넌트를 렌더링하는 것이 가능하다.

```jsx
'use client';
 
import { useEffect } from 'react';
 
export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Optionally log the error to an error reporting service
    console.error(error);
  }, [error]);
 
  return (
    <main className="flex h-full flex-col items-center justify-center">
      <h2 className="text-center">Something went wrong!</h2>
      <button
        className="mt-4 rounded-md bg-blue-500 px-4 py-2 text-sm text-white transition-colors hover:bg-blue-400"
        onClick={
          // Attempt to recover by trying to re-render the invoices route
          () => reset()
        }
      >
        Try again
      </button>
    </main>
  );
}
```

- **클라이언트 컴포넌트** 여야만 한다는 점과, `useEffect` 훅을 이용하여 에러를 핸들링하는 부분을 눈여겨보면 좋을 듯?

### notFound

- 조금 더 정밀한 에러 핸들링을 원하면 고려할 수 있는 방법. 에러 핸들링을 원하는 특정 상황에서 `notFound()` 함수를 던지고, 경로에 `not-found.tsx` 파일을 통해 해당 에러 발생 시 렌더링할 컴포넌트를 설정할 수 있다.

## Chapter 14

### ESLint

- `Next.js` 에 기본적으로 포함된 `eslint-plugin-jsx-a11y` 플러그인들 통해 접근성 에러들을 사전에 찾아내고 방지하는 데에 도움을 받을 수 있다.
- 

## Chapter 15

### **Authentication(인증) vs. Authorization(권한 부여)**

- `Authentication` (인증) - 사용자가 자신이 말한 사람이 맞는지 확인하는 것. 이름과 비밀번호 같은 정보를 통해 신원을 증명하는 것을 의미한다.
- `Authorization` (권한 부여) - 인증의 다음 단계로, 신원이 확인되면 권한 부여를 통해 어플리케이션의 어떠한 부분을 사용할 수 있는지 결정하는 것을 의미한다.

### NextAuth.js

- `Next.js` 에서 제공하는 라이브러리로, 세션 관리, 로그인 및 로그아웃, 기타 인증 측면과 관련된 복잡성을 상당 부분 추상화여 제공함으로써 `Next.js` 어플리케이션에서 인증을 위한 통합 솔루션을 제공해준다.
