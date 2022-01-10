# Relations
+ `one-to-one` using `@OneToOne`
+ `many-to-one` using `@ManyToOne`
+ `one-to-many` using `@OneToMany`
+ `many-to-many` using `@ManyToMany`

## Creat related entity
### Creating a one-to-one relation

```ts
import { Entity, Column, PrimaryGeneratedColumn, OneToOne, JoinColumn } from "typeorm";
import { Photo } from "./Photo";

@Entity()
export class PhotoMetadata {

    @PrimaryGeneratedColumn()
    id: number;

    @Column("int")
    height: number;

    @Column("int")
    width: number;

    @Column()
    orientation: string;

    @Column()
    compressed: boolean;

    @Column()
    comment: string;

    @OneToOne(type => Photo) // @OneToOne decorator
    @JoinColumn()
    photo: Photo;
}
```

`@OneToOne` decorator를 이용해서 두 개의 entity 사이에서 일대일 관계를 설정할 수 있다. 
`type => Photo`는 관계를 만들고자 하는 entity의 클래스를 반환하는 함수이다. (class를 직접적으로 쓰는 것이 아니라)
`() => Photo`로도 작성할 수 있지만, 가독성을 위해서 위와 같이 작성하였다.

#### JoinColum
또한 `@JoinColumn` decorator도 이용할 수 있다. 관계를 소유한다는 걸 나타내는 데코레이터를 추가하는 것이다. 이 데코레이터는 관계의 소유자 측에서 사용한다. 차일드 엔티티에서는 쓰지 않는다. 따라서 Photo가 photo_metadata의 차일드가 되는 것이다.

If you run the app, you'll see a newly generated table, and it will contain a column with a foreign key for the photo relation:
```
+-------------+--------------+----------------------------+
|                     photo_metadata                      |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| height      | int(11)      |                            |
| width       | int(11)      |                            |
| comment     | varchar(255) |                            |
| compressed  | boolean      |                            |
| orientation | varchar(255) |                            |
| photoId     | int(11)      | FOREIGN KEY                |
+-------------+--------------+----------------------------+
```



### Save a one-to-one relation
Now let's save a photo, its metadata and attach them to each other.

```ts
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";
import { PhotoMetadata } from "./entity/PhotoMetadata";

createConnection(/*...*/).then(async connection => {

    // 1. create a photo
    let photo = new Photo();
    photo.name = "Me and Bears";
    photo.description = "I am near polar bears";
    photo.filename = "photo-with-bears.jpg";
    photo.views = 1;
    photo.isPublished = true;

    // 2. create a photo metadata
    let metadata = new PhotoMetadata();
    metadata.height = 640;
    metadata.width = 480;
    metadata.compressed = true;
    metadata.comment = "cybershoot";
    metadata.orientation = "portrait";
    metadata.photo = photo; // this way we connect them

    // 3. get entity repositories
    let photoRepository = connection.getRepository(Photo);
    let metadataRepository = connection.getRepository(PhotoMetadata);

    // 4. first we should save a photo
    await photoRepository.save(photo);

    // 5. photo is saved. Now we need to save a photo metadata
    await metadataRepository.save(metadata);

    // 6. done
    console.log("Metadata is saved, and the relation between metadata and photo is created in the database too");

}).catch(error => console.log(error));
```

#### PhotoMetadata가 테이블에 photo가 속해있기 때문에 DB에 저장되는 순서는 아래와 같다.
1. Photo와 PhotoMetadata가 DB와 연결하고
2. Photo 데이터를 DB에 먼저 저장한 뒤에
3. PhotoMetadata가 DB에 저장한다.

왜냐면 Photo가 PhotoMetadata에 속해있기 때문에 하위 entity를 새로 저장했을 때 자동으로 업데이트가 안되기 때문이다. PhotoMetadata가 Photo PK를 참조하고 있기 때문에 소유자는 PhotoMetadata고, Photo는 PhotoMetadata의 차일드다. 따라서 metadata.entity.ts 파일에서 `@OneToOne(()=> Photo)` 와 `@JoinColmn() user: User` 코드를 작성한다. 

The owner of the relation is PhotoMetadata, and Photo doesn't know anything about PhotoMetadata.
관계 소유자가 PhotoMetadata기 때문에 Photo는 소유자에 대해서 알지 못한다. 이는 Photo 사이드에서 PhotoMetadata에 접근하기 어렵게 만든다. 따라서 아래의 코드와 같이 수정해준다.


```ts
import { Entity, Column, PrimaryGeneratedColumn, OneToOne, JoinColumn } from "typeorm";
import { Photo } from "./Photo";

@Entity()
export class PhotoMetadata {

    /* ... other columns */

    @OneToOne(type => Photo, photo => photo.metadata) // photo => photo.metadata 추가
    @JoinColumn() // @JoinColumn decorator추가
    photo: Photo;
}
```
```ts
import { Entity, Column, PrimaryGeneratedColumn, OneToOne } from "typeorm";
import { PhotoMetadata } from "./PhotoMetadata";

@Entity()
export class Photo {

    /* ... other columns */

    @OneToOne(type => PhotoMetadata, photoMetadata => photoMetadata.photo) // photo => photo.metadata 추가
    metadata: PhotoMetadata;
}
```

`@JoinColumn` decorator는 1대 1 관계에서만 사용한다. 소유 측에서 쓰므로 `@JoinColumn` 데코레이터를 작성한 side에서 관계의 소유권을 가진다.


### Loading objects with their relations
: 관계가 있는 엔티티들의 객체 값 얻기

시퀄라이즈에서는 findOne, findOrCreate 등 자체 메서드를 사용했지만 typeORM은 find 함수보다 더 복잡한 query를 날릴 수 있는 QueryBuilder를 많이 이용한다. 


**find 메서드를 이용했을 때**
```ts
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";
import { PhotoMetadata } from "./entity/PhotoMetadata";

createConnection(/*...*/).then(async connection => {

    /*...*/
    let photoRepository = connection.getRepository(Photo);
    let photos = await photoRepository.find({ relations: ["metadata"] });

}).catch(error => console.log(error));
```

**QueryBuilder를 사용했을 때**
```ts
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";
import { PhotoMetadata } from "./entity/PhotoMetadata";

createConnection(/*...*/).then(async connection => {

    /*...*/
    let photos = await connection
            .getRepository(Photo)
            .createQueryBuilder("photo")
            .innerJoinAndSelect("photo.metadata", "metadata")
            .getMany();


}).catch(error => console.log(error));
```


## Cascade
#### Using cascades to automatically save related objects
: cascade 옵션을 이용해서 연관된 객체들을 자동으로 저장할 수 있다. 
Using cascade allows us not to separately save photo and separately save metadata objects now. 


#### cascade 옵션을 true로 설정
```ts
export class Photo {
    /// ... other columns

    @OneToOne(type => PhotoMetadata, metadata => metadata.photo, {
        cascade: true, // cascade 옵션
    })
    metadata: PhotoMetadata;
}
```


```ts
createConnection(options).then(async connection => {

    // create photo object
    let photo = new Photo();
    photo.name = "Me and Bears";
    photo.description = "I am near polar bears";
    photo.filename = "photo-with-bears.jpg";
    photo.isPublished = true;

    // create photo metadata object
    let metadata = new PhotoMetadata();
    metadata.height = 640;
    metadata.width = 480;
    metadata.compressed = true;
    metadata.comment = "cybershoot";
    metadata.orientation = "portrait";

    photo.metadata = metadata; // this way we connect them

    // get repository
    let photoRepository = connection.getRepository(Photo);

    // saving a photo also save the metadata
    await photoRepository.save(photo);

    console.log("Photo is saved, photo metadata is saved too.")

}).catch(error => console.log(error));
```

방금 우리는 `photo.metadata = metadata` 구문을 통해서 photo's metadata property를 만들었다. cascade는 `photo` side에서 metadata와 연결했을 때만 작동한다. 만약 metadata side에서 photo를 연결했다면 metadata는 자동적으로 저장되지 않을 것이다. 정리하자면, photo_metadata에 photo가 들어가는 것이고(하위 엔티티) 원래는 하위 엔티티를 먼저 저장한 뒤 부모 엔티티를 각자 따로 저장했다면 cascade 옵션을 이용해서 하위 엔티티에 상위 엔티티를 객체 키값으로 넣어 (`photo.metadata = metadata`) photo만 save(`await photoRepository.save(photo)`)해준다면 metadata 까지 자동으로 저장된다. 

Keep in mind - great power comes with great responsibility. Cascades may seem like a good and easy way to work with relations, but they may also bring bugs and security issues when some undesired object is being saved into the database. Also, they provide a less explicit way of saving new objects into the database.

### Cascade Options





### Creating a many-to-one / one-to-many relation
1:N 관계를 만들어보자. 한 명의 작가는 여러 개의 photo를 가질 수 있다. 

**Author class:**
```ts
import { Entity, Column, PrimaryGeneratedColumn, OneToMany, JoinColumn } from "typeorm";
import { Photo } from "./Photo";

@Entity()
export class Author {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @OneToMany(type => Photo, photo => photo.author) // note: we will create author property in the Photo class below
    photos: Photo[];
}
```
작가가 여러개의 작품(photo)을 가지는 것이니 `@OneToMay` decorator를 쓴다. other side of the relation에서 `@ManyToOne` decorator와 무조건 같이 사용해야 된다.


**Photo entity:**
```ts
import { Entity, Column, PrimaryGeneratedColumn, ManyToOne } from "typeorm";
import { PhotoMetadata } from "./PhotoMetadata";
import { Author } from "./Author";

@Entity()
export class Photo {

    /* ... other columns */

    @ManyToOne(type => Author, author => author.photos)
    author: Author; // created author property
}
```
many-to-one / one-to-many relation에서는 the owner side가 항상 many-to-one이다. 
```
+-------------+--------------+----------------------------+
|                          author                         |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
+-------------+--------------+----------------------------+
```

```
+-------------+--------------+----------------------------+
|                         photo                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
| description | varchar(255) |                            |
| filename    | varchar(255) |                            |
| isPublished | boolean      |                            |
| authorId    | int(11)      | FOREIGN KEY                |
+-------------+--------------+----------------------------+
```

table을 살펴보면 photo entity에서 authorId 컬럼이 생성됐고, author의 PK와 연결되어 photo가 owner side가 된다. 

### Creating a many-to-many
N:M 관계를 만들어보자. photo는 여러개의 앨범에 담길 수 있고, 여러개의 앨범 또한 여러개의 photos를 contain할 수 있다.

**Album class:**
```ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable } from "typeorm";

@Entity()
export class Album {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @ManyToMany(type => Photo, photo => photo.albums)
    @JoinTable()
    photos: Photo[];
}
```

**Photo class:**
```ts
export class Photo {
    /// ... other columns

    @ManyToMany(type => Album, album => album.photos)
    albums: Album[];
}
```

**album_photos_photo_albums junction table:**
```
+-------------+--------------+----------------------------+
|                album_photos_photo_albums                |
+-------------+--------------+----------------------------+
| album_id    | int(11)      | PRIMARY KEY FOREIGN KEY    |
| photo_id    | int(11)      | PRIMARY KEY FOREIGN KEY    |
+-------------+--------------+----------------------------+
* Album과 Photo의 PK로 연결된다.
```

`@JoinTable`은 관계의 owner side를 명시하기 위해서 필요하다... 라고 공식 문서에 나와있는데, 가독성을 위해 (또한 설정하기도 더욱 편리하다) `@ManyToMany`와 `@JoinTable()`은 잘 쓰지 않는다. 이유는 [Many-to-Many with custom fields](https://github.com/typeorm/typeorm/issues/1224)를 읽어보면, `@ManyToMany`와 `@JoinTable()`를 쓰게되면 자동으로 조인테이블이 만들어져서 id값만 들어가는데, 조인 테이블에는 id값만 들어가는 게 아니라 여러 다른 컬럼도 들어가야 할 수도 있다. 따라서 `@ManyToMany`와 `@JoinTable()`를 쓰고나서 추가로 더 컬럼 설정을 하는 것보다 애초에 저 둘을 쓰지 않고 `@OneToMany`와 `@ManyToOne`을 이용하여 조인 테이블까지 한꺼번에 설정해준다.

### additional columns in many-to-many table
`@OneToMany`와 `@ManyToOne`을 이용하여 N:M 관계 만드는 예시
```ts
@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number;
    @Column()
    name: string;
    @OneToMany(type => UserGroup, userGroup => userGroup.user)
    userGroups: UserGroup[];
}
@Entity()
export class UserGroup {
    @Column()
    isActive: boolean;
    @ManyToOne(type => User, user => user.userGroups, { primary: true })
    user: User;
    @ManyToOne(type => Group, group => group.userGroups, { primary: true })
    group: Group;
}
@Entity()
export class Group {
    @PrimaryGeneratedColumn()
    id: number;
    @Column()
    name: string;
    @OneToMany(type => UserGroup, userGroup => userGroup.group)
    userGroups: UserGroup[];
}
```

#### @ManyToMany와 @Jointable()을 사용했을 때 작성해야되는 코드
```ts
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number

  @Column()
  name: string

  @OneToMany(type => UserGroup, userGroup => userGroup.user)
  userGroups: UserGroup[];

  @ManyToMany(type => Group, group => group.users)
  @JoinTable({
    name: 'user_groups',
    joinColumn: {
      name: 'user_id',
      referencedColumnName: 'id',
    },
    inverseJoinColumn: {
      name: 'group_id',
      referencedColumnName: 'id',
    },
  })
  groups: Group[]
}

@Entity('user_groups')
export class UserGroup {
  @Column()
  isActive: boolean

  @JoinColumn("user_id")
  @ManyToOne(type => User, user => user.userGroups, { primary: true })
  user: User

  @JoinColumn("group_id")
  @ManyToOne(type => Group, group => group.userGroups, { primary: true })
  group: Group
}

@Entity()
export class Group {
  @PrimaryGeneratedColumn()
  id: number

  @Column()
  name: string

  @ManyToMany(type => User, user => user.groups)
  users: UserGroup[]

  @OneToMany(type => UserGroup, userGroup => userGroup.group)
  userGroups: UserGroup[];
}
```

위의 두 코드를 비교하면 확연한 차이를 느낄 수 있다.
