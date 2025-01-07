---
title: "【Kloudハッカソン#5】開発未経験二人を引き連れてハッカソンに突撃する"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [
    "ハッカソン",
    "アプリ開発",
    "React",
    "Firestore",
    "tailwindcss"
]
published: false
---
# はじめに
2025年1月5日に開催された，「Kloudハッカソン#5」にクラスメート二人を誘って参加しました．
Kloudハッカソン#4では，開発経験のない友人を一人誘って参加したのですが，
今回は**開発未経験２名**にバージョンアップしております．
この時点で私がテックリーダーを担うことがほぼ確定しました
# イカれたメンバーを紹介
- **Nimono**
    - 俺
    - チームリーダー兼テックリーダー
    - 友人と遊んでいたARKにて14時間ぶっ続けで拠点を建てた
    - ハッカソン参加は３回目
- **Haru**
    - 友人A
    - 主にコンポーネント制作で参加
    - テスト前日に卓球の大会入れる人
    - 開発初参加
- **Atok**
    - 友人B`
    - イラスト担当
    - 冬休みをLoLに溶かした
    - 開発初参加
# 構想
まずは作るものを決めなければとアイデアを出していきます．
今回のテーマは **「新年を楽しむ」** でした．
- 新年はこたつに入っていることが多いな
- 家でこたつに入っていると友人と新年を過ごすことができないな

**じゃあこたつに入りながら話せるチャットアプリがあったらいいんじゃね？**

てことでこたつに入りながら落ち着いた時間を過ごせるチャットアプリ
**「こたつメタバース」** を作ることが決まりました．

# 技術選定

|フロントエンド||バックエンド|DB|その他|
|---|---|---|---|---|
|[![](https://img.shields.io/badge/-Node.js-000000.svg?logo=node.js&style=for-the-badge)](https://nodejs.org/en/)|[![](https://img.shields.io/badge/-React-000000.svg?logo=react&style=for-the-badge)](https://ja.react.dev/)|![](https://img.shields.io/badge/-Javascript-000000.svg?logo=javascript&style=for-the-badge)|[![](https://img.shields.io/badge/-Firestore-000000.svg?logo=firebase&style=for-the-badge)](https://firebase.google.com/docs/firestore?hl=ja)|[![](https://img.shields.io/badge/-tailwindcss-000000.svg?logo=tailwindcss&style=for-the-badge)](https://tailwindcss.com/)|

フロントにはもともと少し知識のあったReactを，
DBにはリアルタイムなデータ更新に特化したfirestoreを，
バックエンドにはfirestoreを手軽に動かすためにJavaScriptを採用しました．
# 設計
私は過去に二度ハッカソンに参加したことがあり，それ以外にも共同開発は数回やってきたのですが，毎回と言っていいほど仕事量・役割分担などの面で問題が生じていました(最強の友人がほぼすべてやってくれたことはありましたが...)

そこで今回は，
**設計をしっかり固めてから開発に取り掛かろう**
と思い立ちました．

その旨をチームメンバーに伝え，コーディングには手を付けず設計から始めました．

## 機能を列挙
まずは具体的にどのような機能を持ったプロダクトになるのか，チームメンバー全員で案を煮詰めていきます．
- 同室内のユーザー間のチャット機能
- ユーザーが自由に部屋を作成可能
- こたつに入れる
- 日時・時刻表示のモニター画面中央上に配置
- ユーザーの移動
  - WASD
    - 自由に移動できる
    - 動きが機械的になりそう
  - マウス追従
    - WASDに同じく
    - 計算がだるそう
  - ランダム
    - 生物感が出そう
    - ユーザーは操作できない

## DB構成
「firestoreはDBの作り方が無限大だからメンバー間で共有しないと爆発するよ」という友人からの助言を受け，DB構成はより時間をかけて設計をしました．
~~結局私しかDBを触らなかったのでそこまで意識しなくてよかったかもですね~~
:::details DB構成
- rooms
  - room1
    - users
      - vSSb7koAKMQ6SpaDnoIT
        - name: str
        - positionInKotatsu
      - user2
      - ...
    - chatlog
      - message
        - content: str
        - timestamp: Date
        - authID: string
    - peopleInkotatsu: int
    - existNorth: bool
    - existEast: bool
    - existWest: bool
    - peopleInRoom: int
  - room2
  - ...
- usersInLobby
  - user1
    - name: str
  - user2
  - ...
:::

ひとまずはこんなところでしょうか．
テキストだけではチームメンバー間で共通した認識を持つことは難しいと思い，簡易的なイラストも制作しました．
![](https://storage.googleapis.com/zenn-user-upload/015b23f94275-20250108.png)
*私が描いた美しい設計図*
シンプルながら，そこかしこに絵心を感じる素晴らしい設計図だと自負しております．

ほかにも，書ききれませんがかなりの時間をかけて設計を行いました

# 開発
サイトに入ると以下の手順を踏むことになります

1. 自身の名前を入力
2. 部屋を作成，あるいは既にある部屋に入室する
3. こたつルームでのひと時を楽しむ

表示するページをコンポーネントとして作成し，ReactRouterで切り替える方式をとりました．

```jsx:App.jsx
import { useState } from 'react'
import './App.css'
import {BrowserRouter, Route, Routes, Link} from "react-router-dom"

//import component 
import Home from './views/Home'
import Signin from './views/Signin'
import SelectRoom from './views/SelectRoom'
import NotFound from './views/NotFound'


function App() {
  const [data, setData] = useState('');
  const [user, setUser] = useState('');

  return (
    <>
      <div className='App'>
        <BrowserRouter>
          <Routes>
            <Route path='/' element={<Signin setUser={setUser}/>}></Route>
            <Route path='/Home' element={<Home user={user} id={data}/>}></Route>
            <Route path='/SelectRoom' element={<SelectRoom user={user} setData={setData}/>}></Route>
            <Route path='*' element={<NotFound/>}></Route>
          </Routes>
        </BrowserRouter>
      </div>
    </>
  )
}

export default App

```

## 名前入力画面
Signinコンポーネントとしました．
DBにユーザー名を含めたユーザーデータを追加し，そのドキュメントをApp.jsxに返します．

HTMLのinputタグに入力された値をEnterキーで返すシステムをHaru君が作ってくれました．
ここ以外の箇所で似たような仕組みが必要になったのですが，Haru君が作ったシステムをそのまま流用させてもらったりしました．めっちゃ感謝．

![](https://storage.googleapis.com/zenn-user-upload/a67494085b95-20250108.png)
*実際の画面*

```jsx:Signin.jsx
import React, { useState } from 'react';
import { db } from '../../firebaseConfig';
import "../index.css";
import { addDoc, collection } from 'firebase/firestore';
import { useNavigate } from 'react-router-dom';

const Signin = (props) => {
    const [userName, setUsername] = useState('');

    const navigate = useNavigate();

    const registerUserNames = () => {
        if (userName.trim() !== '') {

            const usersCollectionRef = collection(db, 'usersInLobby');
            addDoc(usersCollectionRef, {
                name: userName,
            })
            .then(docRef => {
                props.setUser(docRef.id);
                navigate('/SelectRoom');
            })
            setUsername(''); // 入力欄をクリア
        }
    };
    const handleKeyDown = (event) => {
        if (event.key === 'Enter') {
            registerUserNames();
        }
    };

    return (
        <div className='font-Koruri flex w-screen h-screen bg-[#847f68] items-center justify-center'>
            <div className='flex w-11/12 h-5/6 bg-[#858585] items-center justify-center'>
                    <div className='text-center'>
                        <h1 className='text-center text-white text-7xl leading-loose font-extrabold'>
                        こたつメタバースへようこそ！
                        </h1>
                        <h1 className='text-center text-white text-7xl leading-loose font-extrabold'>
                        あなたの名前を入力してください
                        </h1>
                        <input
                            className='text-center w-11/12 h-24 mt-4 p-2 text-5xl font-bold rounded-lg transition duration-500 ease-in-out transform hover:scale-105'
                            type="text"
                            value={userName}
                            onChange={(e) => setUsername(e.target.value)}
                            onKeyDown={handleKeyDown} 
                        />
                    </div>
            </div>
        </div>
    );
};

export default Signin;
```

## 部屋選択画面
SelectRoomコンポーネントとしました．
Userのドキュメントを引数に，Roomドキュメント配下のUsersコレクションにドキュメントを追加し，Home.jsxに画面を遷移します．

画面内には
- Signinに戻るホームボタン
- 現在存在する部屋の一覧を表示・クリックすると入室できるRoomボタン
- 新しく部屋を作成するRoom作成ボタン

があります．

![](https://storage.googleapis.com/zenn-user-upload/249956c2afe7-20250108.png)

```jsx:SelectRoom.jsx
import { collection, doc, getDoc, onSnapshot, orderBy, query } from "firebase/firestore";
import { db } from "../../firebaseConfig"
import { React, useState, useEffect } from "react"
import { useNavigate } from "react-router-dom";

import RoomButton from "../components/RoomButton";
import CreateRoom from "../components/CreateRoom";

import "../index.css";

const SelectRoom = (props) => {

    const [currentUserName, setCurrentUserName] = useState('');

    const [rooms, setRooms] = useState([]);

    const [isShowModal, setIsShowModal] = useState(false);

    const navigate = useNavigate();

    useEffect(() => {

        const usersDocumentRef = doc(db, 'usersInLobby', props.user);
        getDoc(usersDocumentRef).then((doc) => {
            if (doc.exists()) {
                setCurrentUserName(doc.data().name);
            } else {
                console.log('No such document!');
            }
        });

        const roomsCollectionRef = collection(db, 'rooms');
        const q = query(roomsCollectionRef, orderBy('peopleInRoom', 'desc'));
        const unsub = onSnapshot(q, (QuerySnapshot) => {
            setRooms(
                QuerySnapshot.docs.map(
                    (doc) => ({ ...doc.data({ serverTimestamps: "estimate" }), id: doc.id })
                )
            )
        });
        return unsub;
    }, []);

    return (
        <div className='font-SourceHanSansJP flex w-screen h-screen bg-amber-100 items-center justify-center'>
            <button onClick={() => navigate("/")} className=" absolute top-5 left-32 w-16 h-16">
                <img src="/Home.svg" alt="home" />
            </button>
            <div className="flex absolute top-5 left-3/4 items-center">
                <div>
                    <img className="w-16 h-16 pr-3" src="/user_logo.svg" alt="room" />
                </div>
                <div>
                    <p className="text-black text-5xl font-extrabold">{currentUserName}</p>
                </div>
            </div>
            <div className="grid gap-4 grid-cols-[repeat(auto-fill,minmax(theme(spacing.60),1fr))] p-10 w-11/12 h-5/6 bg-white">
                {rooms.map((room) => (
                    <RoomButton key={room.id} user={props.user} room={room} setData={props.setData} />
                ))}
                <button onClick={() => setIsShowModal(true)} className="flex w-32 h-32 font-Koruri text-white text-[96px] font-normal bg-blue-800 absolute rounded-full bottom-36 right-36 items-center justify-center transition duration-200 ease-in-out transform hover:scale-110">+</button>
                {isShowModal && (
                    <CreateRoom user={props.user} setIsShowModal={setIsShowModal} />
                )}
            </div>
        </div>
    )
}

export default SelectRoom
```

## 部屋画面
Homeコンポーネントとしました．
User・Roomのドキュメントを引数として，
こたつ，時計，チャットなど各種コンポーネントを表示しています．

ドット絵全般をAtok君にやってもらいました．
ドット絵の経験はなかったとのことですが，そうとは感じさせないクオリティのイラストを制作してくれました．ありがとう！

時計コンポーネントをHaru君が担当してくれました．
初めて触るJavaScript・CSSを独学で勉強し，私がチーム結成当初想像していた何倍も仕事をしてくれました．正直相当助けられました．


![](https://storage.googleapis.com/zenn-user-upload/1b92249da87b-20250108.png)

```jsx:Home.jsx
import React from 'react'
import Chat from '../components/Chat'
import Kotatsu from '../components/Kotatsu'
import Monitor from '../components/Monitor'
import LeaveRoomButton from '../components/LeaveRoomButton'

const Home = ({ user, id }) => {

    return (
        <div className='w-screen h-screen flex flex-col items-center justify-center'>
            <div className='w-full h-full'>
                <img className='w-full h-full' src="/room.png" alt="room" />
            </div>
            <div className='absolute top-10 right-10'>
                <LeaveRoomButton roomID={id} userID={user} />
            </div>
            <div className='absolute top-16 w-1/3 h-1/3'>
                <Monitor />
            </div>
            <div className='scale-[30%] absolute -bottom-16'>
                <img src="user_north.png" alt="" />
            </div>
            <div className='scale-[30%] absolute -bottom-32 right-[760px]'>
                <img src="user_east.png" alt="" />
            </div>
            <div className='scale-[30%] absolute -bottom-32 left-[760px]'>
                <img src="user_west.png" alt="" />
            </div>
            <div className='absolute bottom-52'>
                <Kotatsu></Kotatsu>
            </div>
            <div className='absolute left-0 bottom-20'>
                <Chat user={user} roomID={id}></Chat>
            </div>
        </div>
    )
}

export default Home
```

## その他コンポーネント
**InputChat.jsx**
入力されたテキスト・送信主・送信時刻を含んだオブジェクトをmessagesコレクションに追加します．

**ChatLog.jsx**
Messagesに保存されているドキュメントの一覧を表示します．

**Clock.jsx**
PCから現在日時を取得し，良い感じにフォーマットして背景の上に重ねて表示します．

**LeaveRoomButton**
Roomから自分のドキュメントを削除し，SelectRoomに画面遷移します．

**CreateRoom.jsx**
Roomの名前・入室人数(初期値:0)・Roomの制作者を含めたオブジェクトをRoomsコレクションに追加します．

**RoomButton.jsx**
Roomの名前・作成者・現在の室内人数が表示されており，クリックするとHomeに遷移します．また，カーソルを合わせるとゴミ箱マークが表示され，押すとRoomを削除できます．
![](https://storage.googleapis.com/zenn-user-upload/ebec5f73902d-20250108.png)

**Kotatsu.jsx**
こたつをクリックすると，こたつに入ることができ，こたつのどの箇所に入るかを決めるシステムをHaru君に作ってもらいましたが，その他の実装が間に合いませんでした．Sorry Haru.

# 反省
## よかった点
- 設計を固めたおかげで開発がスムーズだった
- コーディング班の役割分担がしっかりとなされ，一人に仕事が集中しなかった．
  - もちろん差はありましたが開発経験の違いをみれば許容範囲内だと思います
- 最終的になんとか形になった
- 一か月でReact・JS・tailwindcss・firestoreの知識がついた
## 改善点
- Userの移動・こたつに入る処理等が未完成
- ドット絵という全く未知の仕事を一人にやらせてしまった
- テックリードとしてもう少しチームメイトに指導をすべきだった
- 開発に集中するあまり，発表スライドの内容に関する指示にミスがあった(気がする)
- サーバーにマウントすることができず，いまだに公開できていない(2025/1/8時点)

# 結果
残念ながら賞をいただくことは叶いませんでした．
反省の章にも書いた通り，まだまだリーダーを務めるには未熟で，私がもっとしっかりしていれば何らかの賞は受賞できていたと思っています．
チームメンバーはたくさん働いてくれたので，受賞を経験させてあげられなかったのは不甲斐なく感じます．

ただ，初めてのテックリードはいろいろなことが勉強でき，大きく成長することができたように思います．

チームメンバーのお二人，急な誘いからここまで付き合ってくれてありがとうございました！

# リンク
https://github.com/Nimono-sleep-well/Kotatsu-Metaverse
https://github.com/Nimono-sleep-well
https://x.com/Nimono_blend