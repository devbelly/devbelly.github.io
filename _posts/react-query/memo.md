client state

web browser session과 관련이 있다.

언어선택
다크모드 라이트모드

특정유저에 대한 상태

server state

서버에 저장된 데이터, user에게 보여져야 한다

리액트 쿼리는 클라이언트에 서버데이터에 대한 캐시를 유지한다, 리액트코드가 서버에 대한 정보를 필요로 한다면 바로 서버로 가는 것이 아니라 대신 react query cache에 질의한다

리액트 쿼리는 데이터를 관리
우리가 해야할 일은 언제 캐시를 업데이트할지 서버로부터온 새로운 데이터에 대해

쿼리 클라이언트?
쿼리들과 캐시를 관리하는 클라이언트
캐시에 대한 정보와 default option을 담고 있다
방법1)
방법2)

캐시구조(key + data)

추가적으로 loading과 error state를 매 쿼리마다 저장하고 있어 직접 관리할 필요가 없다.

pagination / infinite scroll

prefrecting

mutation

deduplicate

retry on error

callback

create query client

apply queryprovider
provider cache and client config to children
value로 queryclient를 갖는다.

usequery 실행

리액트 쿼리의 결과는 많은 오브젝트로 이루어져있다.
https://react-query-v3.tanstack.com/reference/useQuery
usequery으 ㅣ첫번째 파라미터는 query key
두번째 파라미터는 query function, 어떻게 데이터를 얻을 것이다.

```
const queryClient = new QueryClient();

function App() {
  return (
    // provide React Query client to App
    <QueryClientProvider client={queryClient}>
      <div className="App">
        <h1>Blog Posts</h1>
        <Posts />
      </div>
    </QueryClientProvider>
  );
}

export default App;
```

```
  const { data } = useQuery("posts", fetchPosts);

```

실행하면 오류가 발생, 나중에 더 나은 방법 배운다.

---

데이터가 없을때 어떻게 처리해야할까?

데이터가 없을떄 indicator사용 is loading is error

isloading vs isfetching

isFetching?
async query function이 resolved 되지 않았다

is Loading?
캐시된 데이터가 없다 + isFetching, 더 좁은 범위이다.
pagination에서 차이를 확연히 확인할 수 있다.

iserror 확인해보기

실패하게 되면 세번 실행한다., 설정가능하다.
기본적으로는 세번 실행하고 데이터를 얻을수 없다고 판단한다.

error 오브젝트도 있다.

usequery의 옵션으로 error call back을 파라미터로 전달할 수 있따.
나중에 할거임.

---

stale time
devtools에서 보면 stale time으로 즉시 변하게 된다.
stale data의미?
data 는 stale data에 대해서만 refetch 된다.

compoenet remount, window focus시 다시 refetch 시도

stale time 즉 data가 생존할수 있는 시간과 같다.

usequery 세번쨰 파라미터는 옵션
{staleTime: 2000}

dev툴엣 어떻게 보이는지 살펴보자.
fresh -> stale로 변경됨을 확인할수 있다.

왜 stale time이 0일까?

stale time은 refetching을 위한
cache는 나중에 필요한 데이터

? no active use query -> data goes cold storage
캐시타임이 지나면 캐시가 만료된다.
기본적으로 5분

캐시 만료되면 garbage collected되고 clinet는 사용핤 ㅜ없다

캐시가 있다면 패치도중에 사용될수 있다.

---

comments를 가져오면 제대로 안가져와지는 문제가 있다.

이유는 쿼리키 때문이당

이미 알고 있는 키에 대한 쿼리는 아래 해당사항이 이쓸때만 refetch한다.

- remount component
- refocus window
- manually run refetch funtion
- automated refetch
- mutation 이후에 invalidate query,즉 클라이언트 데이터가 서버 데이터와 일치하지 않음을 나타낸다.

즉 데이터가 stale이더라도 refetch가 이루어지지 않는다.

solution?

쿼리 키가 id를 포함하게 한다.
배열을 전달하면 가능해진다
['commnets, post.id]

---

next , prev 버튼 만들기. pagination으로 가능하다.

이전에 comments 예시에서 살펴보았듯이, page number을 쿼리키에 포함할 필요가 있다.

구현

페이지 넘어갈때 fetching..를 방지해야함

---

prefetching

method of queryClient

keepPreviousdata: true

---

isLoading vs isFetching

isFetching true
async function hasn't yet resolved

isLoading
isFetching + no cached data for the query

---

mutation
서버 데이터를 업데이트하는 쿼리
add delete edit

업데이트
cache 업데이트
invalidate query - > trigger refetch

useMutation은 usequery오 ㅏ비슷하지만 mutation function을 리턴한다. 이 함수를 통해 servercall을 한다.
쿼리키는 피료없다. isLoading, 있고 isFetching 없다
이는 mutation과 관련된 cache가 없기 때문이다.
기본적으로 retry 하지 않는다.
