인스타그램은 사진을 업로드하고, 사람들이 서로 댓글을 달며 서로 follow 관계를 만들 수 있는 사진 SNS 서비스이다. 내가 인스타그램의 초기 개발팀에 속해 있었다고 가정해보겠다. 다행히도 모든 기능과 프로토타입은 완성되어 있다. 백엔드 개발자로서 데이터베이스의 스키마를 디자인해야 한다면, 어떻게 스키마 디자인을 하는 게 좋을까? 


# 주요 기능
1. 게시물(Post) 작성 기능
인스타그램에서는 여러 개의 사진(A)을 올릴 수 있습니다. 사진을 업로드할 때, 사진을 설명하는 간단한 글(C)이 올라갑니다.

2. 게시물에 댓글 달기 및 좋아요 기능
게시물이 업로드되면 다른 사용자는 이 게시물에 댓글(E)을 달 수 있고, 좋아요(B) 를 눌러 관심을 표할 수 있습니다.

3. 해시태그 기능
게시물에 #감성 #맛집 등의 해시태그(D)를 남길 수 있으며, 이 해시태그를 누르면 이 해시태그가 사용된 모든 게시물을 한 데 모아 볼 수 있습니다.

4. follow 기능
인스타그램에서 친구 관계는 팔로워(follower)와 팔로잉(following)으로 나뉩니다. 김코딩이 최해커를 following 하면, 최해커의 피드가 김코딩의 홈 화면에 나타납니다. 최해커의 입장에서는 김코딩이 follower로 추가됩니다.



## Database 설계

#### user : post = 1 : N
한 명의 user는 여러개의 post를 작성할 수 있다.
하나의 post는 하나의 유저만 작성할 수 있다.

#### user : follow = 1 : N
한 명의 user는 여러명을 follow 할 수 있다.

#### post : media = 1 : N
하나의 post에 여러 개의 media를 올릴 수 있다.

#### post : text = 1 : 1
하나의 post는 하나의 textcontent만 작성할 수 있다.
하나의 textcontent는 하나의 post에만 적용된다.

#### post : like = 1: N
하나의 post는 여러개의 like를 받을 수 있다.

#### post : comment = 1 : N
하나의 post는 여러개의 댓글이 달릴 수 있고
여러개의 댓글은 하나의 post에 존재한다.

#### post : hashtag = 1 : N
하나의 post는 여러개의 해시태그를 가질 수 있다.


## 코드 작성


```js
CREATE TABLE `user` (
  `id` INT PRIMARY KEY AUTO_INCREMENT,
  `username` varchar(255)
);


CREATE TABLE `follow` (
  `id` INT PRIMARY KEY AUTO_INCREMENT,
  `following_id` varchar(255),
  `follower_id` varchar(255)
);


CREATE TABLE `post` (
  `id` INT PRIMARY KEY AUTO_INCREMENT,
  `user_id` INT,
  `post_text` varchar(255),
  `created_at` datetime DEFAULT (CURRENT_TIMESTAMP)
);


CREATE TABLE `hashtag` (
  `id` INT PRIMARY KEY AUTO_INCREMENT,
  `post_id` INT,
  `comment_id` INT,
  `hashtag_txt` varchar(255),
  `hashtag_count` varchar(255)
);

CREATE TABLE `like` (
  `id` INT PRIMARY KEY AUTO_INCREMENT,
  `user_id` INT,
  `post_id` INT,
  `like_count` INT
);

CREATE TABLE `comment` (
  `id` INT PRIMARY KEY AUTO_INCREMENT,
  `comment_txt` varchar(255),
  `user_id` INT,
  `post_id` INT,
  `comment_count` INT,
  `comment_like` INT,
  `created_at` datetime DEFAULT (CURRENT_TIMESTAMP)
);

CREATE TABLE `media` (
  `id` INT PRIMARY KEY AUTO_INCREMENT,
  `user_profil_photo` varchar(255),
  `post_photo` varchar(255)
);



ALTER TABLE `follow` ADD FOREIGN KEY (`following_id`) REFERENCES `user` (`id`);
ALTER TABLE `follow` ADD FOREIGN KEY (`follower_id`) REFERENCES `user` (`id`);
ALTER TABLE `post` ADD FOREIGN KEY (`user_id`) REFERENCES `user` (`id`);
ALTER TABLE `like` ADD FOREIGN KEY (`user_id`) REFERENCES `user` (`id`);
ALTER TABLE `comment` ADD FOREIGN KEY (`user_id`) REFERENCES `user` (`id`);
ALTER TABLE `media` ADD FOREIGN KEY (`user_profil_photo`) REFERENCES `user` (`id`);

ALTER TABLE `hashtag` ADD FOREIGN KEY (`post_id`) REFERENCES `post` (`id`);
ALTER TABLE `media` ADD FOREIGN KEY (`post_photo`) REFERENCES `post` (`id`);
ALTER TABLE `comment` ADD FOREIGN KEY (`post_id`) REFERENCES `post` (`id`);
ALTER TABLE `like` ADD FOREIGN KEY (`post_id`) REFERENCES `post` (`id`);

ALTER TABLE `hashtag` ADD FOREIGN KEY (`comment_id`) REFERENCES `comment` (`id`);

```