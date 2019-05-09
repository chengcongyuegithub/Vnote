# timeLine新鲜事(一)
## 功能简介
一个新鲜事,在这个项目中表示XXXX关注了某个问题,XXXX回答了某个消息,它是一个feed(model),我们来看一下
```
    private int id;
    private int type;
    private int userId;
    private Date createDate;
    private String data;
    private JSONObject dataJSON=null;
```
id 对应唯一非空的主键,type表示是评论还是关注
userId做这个动作的人,创建时间,data,JSONObject一会在看.
然后就是DAO层

## 推拉模式
从dao层就开始分推模式和拉模式
在了解这两个模式之前,还是最简单的添加新鲜事
```
 @Insert({"insert into ",TABLE_NAME,"(",INSERT_FIELDS,")" +
            "values (#{userId},#{data},#{createdDate},#{type})"})
    int addFeed(Feed feed);
```
```
 @Select({"select ",SELECT_FIELDS," from ",TABLE_NAME," where id=#{id}"})
 Feed getFeedById(int id);
```
```
List<Feed> selectUserFeeds(@Param("maxId")int maxId,@Param("userIds")List<Integer> userIds,
                               @Param("count")int count);
```
这个SQL比较复杂,我们在XML中完成
```
<mapper namespace="com.ccy.easyzhihu.Dao.FeedDAO">
    <sql id="table">feed</sql>
    <sql id="selectFields">id,create_date,user_id,data,type
    </sql>
    <select id="selectUserFeeds" resultType="com.ccy.easyzhihu.model.Feed">
        SELECT
        <include refid="selectFields"/>
        FROM
        <include refid="table"/>

        WHERE id &lt; #{maxId}
        <if test="userIds.size!=0">
             AND user_id in
            <foreach item="item" index="index" collection="userIds"
                     open="(" separator="," close=")">
                #{item}
            </foreach>
        </if>
        ORDER BY id DESC LIMIT #{count}
    </select>
</mapper>
```


## Controller层的实现
以下就是拉模式的实现:
```
@RequestMapping(path = {"/pullfeeds"},method = {RequestMethod.GET,RequestMethod.POST})
   private String getPullFeeds(Model model)
   {
       int localUserId=hostHolder.getUser()!=null?hostHolder.getUser().getId():0;
       List<Integer> followees=new ArrayList<>();
       if(localUserId!=0)
       {
          followees=followService.getFollowees(localUserId, EntityType.ENTITY_USER,Integer.MAX_VALUE);
       }
       List<Feed> feeds =feedService.getUserFeeds(Integer.MAX_VALUE,followees,10);
       model.addAttribute("feeds",feeds);

       return "feeds";
   }
```
拉的模式,先获取用户,看看是否登录,如果用户不为空,那么就获取用户关注的人,然后执行getUserFeeds从所有的关注的人那里获取新鲜事,这些新鲜是当然是在数据库的,那么它是如何添加如数据库的.这就是异步队列实现的功能了我们一会再去看异步队列,我们先来按一下**推模式**
```
 @RequestMapping(path = {"/pushfeeds"},method = {RequestMethod.GET,RequestMethod.POST})
   public String getPushFeeds(Model model)
   {
       int localUserId=hostHolder.getUser()!=null?hostHolder.getUser().getId():0;
       List<String> feedIds=jedisAdapter.lrange(RedisKeyUtil.getTimelineKey(localUserId),0,10);
       List<Feed> feeds=new ArrayList<>();
       for (String feedId:feedIds)
       {
           Feed feed = feedService.getById(Integer.parseInt(feedId));
           if(feed!=null)
           {
               feeds.add(feed);
           }
       }
       model.addAttribute("feeds",feeds);
       return "feeds";
   }
```
## 异步队列的实现
在是comment,和follow的时候,会使用feedHandler
```
@Override
    public List<EventType> getSupportEventTypes() {
        return Arrays.asList(new EventType[]{EventType.COMMENT,EventType.FOLLOW});
    }
```
然后就是实现,
```
  @Override
    public void doHandler(EventModel model) {
        Random r=new Random();
        model.setActorId(1+r.nextInt(10));
        //pull
        Feed feed=new Feed();
        feed.setCreateDate(new Date());
        feed.setType(model.getType().getValue());
        feed.setUserId(model.getActorId());
        feed.setData(buildFeedData(model));
        if(feed.getData()==null)
        {
            return ;
        }
        feedService.addFeed(feed);
        //push
        List<Integer> followers=followService.getFollowers(EntityType.ENTITY_USER,
                model.getActorId(),Integer.MAX_VALUE);
        followers.add(0);
        for(int follower:followers)
        {
            String timelineKey= RedisKeyUtil.getTimelineKey(follower);
            jedisAdapter.lpush(timelineKey,String.valueOf(feed.getId()));
        }

    }
```
创建一个新鲜事,然后填入日期,填入类型是follow还是comment,填上发起的人,填如model的**真实数据**,然后向数据库中添加这个新鲜事,这就为拉的模式做好了准备.
然后就是推的模式,得到当前人的所有的关注人,然后添加一个系统队列,也就是0,然后遍历,把这个放入到redis的list中去.

上面就是实现.
对于真实数据的处理如下:
```
 private String buildFeedData(EventModel model)
    {
       Map<String,String> map=new HashMap<>();
       User actor=userService.getUser(model.getActorId());
       if(actor==null)
       {
           return null;
       }
       map.put("userId",String.valueOf(actor.getId()));
       map.put("userHead",actor.getHeadUrl());
       map.put("userName",actor.getName());
       if(model.getType()==EventType.COMMENT
               ||
       (model.getType()==EventType.FOLLOW
       &&model.getEntityType()== EntityType.ENTITY_QUESTION))
       {
           Question question=questionService.getQuestionById(model.getEntityId()+"");
           if(question==null)
           {
               return null;
           }
           map.put("questionId",String.valueOf(question.getId()));
           map.put("questionTitle",question.getTitle());
           return JSONObject.toJSONString(map);
       }
       return null;
    }
```
对真实数据的处理就是讲model转化成json.
将发起人放入,讲发起人的信息放入,判断model的类型是comment还是follow,得到作用的问题.然后将问题放入到map中.
上面就是基本的实现

![](_v_images/20190501101227641_1043920637.png =856x)
![](_v_images/20190501101745302_474376946.png =806x)
