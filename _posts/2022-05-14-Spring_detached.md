---
title: 
    "[JPA] detatched entity passed to persist 영속성 에러"
excerpt: 
    
category: 
    JPA
    Spring
tags: 
    에러
    예외
toc: 
    true
comments: 
    true
use_math: 
    false
    #for use, there must be dollar sign on both side of contents
---

<style type = 'text/css'>
    .o{
    font-weight: bold;
    color:orange;
    }
</style>

## 개요  
스프링을 배운 얕은 지식으로 프로젝트를 진행해보고자 테스트 DB인 h2와 연결하여 엔티티들이 잘 만들어지는지 확인했다.  
먼저 사용한 클래스들을 나열하자면 다음과 같다.

```java  
@Getter
@ToString
@Entity
public class Post{
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "post_id")
    private Long post_id;

    private Date posted_Data;
    private String post_Owner;
    private String post_Title;
    private String contents;
    private String category;
    }
```
```java
@Repository
@RequiredArgsConstructor
public class BoardRepositoryImpl implements BoardRepository{

    @PersistenceContext
    private final EntityManager em;

    @Override
    public Long save(Post post) {
        em.persist(post);
        return post.getPost_id();
    }

    @Override
    public Post findOne(Long id) {
        return em.find(Post.class, id);
    }

    @Override
    public List<Post> findAll() {
        return em.createQuery("select i from Post i", Post.class).getResultList();
    }
}
```  
```java
@RequiredArgsConstructor
@Service
@Transactional(readOnly = true)
public class BoardServiceImpl implements BoardService{

    private final BoardRepository boardRepository;

    @Override
    @Transactional
    public Long savePost(Post post) {
        boardRepository.save(post);
        return post.getPost_id();
    }

    @Override
    public List<Post> findAllPost() {
        return boardRepository.findAll();
    }

    @Override
    public Post findOne(Long postId) {
        return boardRepository.findOne(postId);
    }

}
```  
## 원인 & 해결
먼저 Post 클래스를 객체로 만들어 이를 db에 persist 하고했는데, detached entity passed to persist오류가 등장했다.  

여기서 Post 클래스안의 <span class = "o"> Id 어노테이션은 JPA가 직접 Id를 생성해주는 역할을 하는데 이 과정에서 JPA는 Post객체를 영속화</span> 시키게된다. <span class = "o"> detached 객체는 예전에 한번 이미 영속화가 된 적 있는 객체</span>이기 때문에 이후로 다시 persist를 시도하면 해당 에러가 나온다.  


```java
@SpringBootTest
class BoardRepositoryTest{

    @Autowired
    private BoardRepository boardRepository;
    @Autowired
    private BoardService boardService;

    @Test
    public void BoardRepositoryTest() throws Exception {
        //given
        Post post = new Post();
        // post.setPost_id(1L);
        post.setPost_title("title");

        //when
        boardService.savePost(post);

        //then

    }
```
위 테스트 에서는 이미 post의 id를 자동생성하겠다는 @Id를 지정하고 persist를 했다. setter를 통해 id가 아닌 title을 set 해줌으로써 해당 에러가 나오지 않게 됐다.
  
추가로 @Id를 통해서 <span class = "o"> 이미 영속화 된 객체를 사용하기 위해서는 persist가 아닌 merge를 이용</span>해야한다.

## 참고
[https://www.inflearn.com/questions/121326]("https://www.inflearn.com/questions/121326")