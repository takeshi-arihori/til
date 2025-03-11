# React×MUIで作る！複数のアコーディオンをスクロールにあわせて追従＆更新する方法
※未完成です。

本記事では、**React** と **MUI (Material-UI)** のアコーディオンコンポーネントを使いつつ、以下のような仕様を実装する方法を紹介します。

1. **複数のアコーディオンを画面上部に追従 (Sticky) させる。**  
2. **上に重なるアコーディオンのサイズが変化したら動的に高さを再計測し、下にあるアコーディオンの `top` 位置を更新する。**  
3. **スクロール位置に応じて、いま画面上部付近にある「セクションタイトル」を状態管理する。**

---

## デモ用データ例 (JSON)

例えば、下記のようなデータを想定してください。  
実際にはバックエンド側から取得するケースを想定しており、深い構造を持つ場合でも応用可能です。

```json
{
  "groupA": {
    "title": "Fruits",
    "description": "Section about various fruits.",
    "items": [
      { "id": 1, "name": "Apple", "detail": "Red or green fruit" },
      { "id": 2, "name": "Banana", "detail": "Yellow fruit" }
    ]
  },
  "groupB": {
    "title": "Vegetables",
    "description": "Section about common vegetables.",
    "items": [
      { "id": 3, "name": "Carrot", "detail": "Orange root vegetable" },
      { "id": 4, "name": "Tomato", "detail": "Red edible fruit/vegetable" }
    ]
  }
}
```

groupA, groupB など複数のグループが存在し、それぞれに title (セクションタイトル) と items (詳細データ) が入っています。

コード全体構成
下記のように 2つのアコーディオン を用意します。

メイン・アコーディオン (上部)。Sticky で追従しつつ、開閉によって高さが変わる。
サブ・アコーディオン (メインの下に追従)。メインのアコーディオンが占有する高さを考慮して、 top を動的に設定。
さらに、スクロール位置に応じて「いまどのセクション(グループ)が画面上部付近にあるか」を調べ、activeSection として状態管理しています。

Styled Components
SCSS の代わりに、Styled Components でレイアウトを定義します。
Sticky な要素を扱うために、position: sticky と top: ...px を指定しています。

```
import styled from "styled-components";

export const Container = styled.div`
  width: 90%;
  margin: 0 auto;
`;

// メインアコーディオン用のラッパ (sticky にして追従)
export const MainAccordionWrapper = styled.div`
  position: sticky;
  top: 0; /* 初期値は 0 とし、後で動的に高さを計測する */
  z-index: 1000;
  background-color: #fff; /* 背景が透過しないよう明示 */
`;

// サブアコーディオン用のラッパ (sticky にして追従)
export const SubAccordionWrapper = styled.div<{ offsetTop: number }>`
  position: sticky;
  top: ${(props) => props.offsetTop}px; /* 動的に設定 */
  z-index: 999;
  background-color: #fff;
`;

// セクション全体を区切る要素
export const SectionBlock = styled.div`
  margin: 40px 0;
  border-bottom: 1px solid #ccc;
`;

// セクションタイトル
export const SectionTitle = styled.h2`
  margin-bottom: 8px;
`;

// スクロール位置を監視するためのブロック (DOM計測用)
export const ScrollSpyBlock = styled.div`
  margin-top: 20px;
`;

```

コンポーネント実装例
下記のコンポーネント例では、

mainAccordionRef でメインアコーディオン全体の DOM を参照。
ResizeObserver で高さを監視し、mainAccordionHeight を常に最新化。
サブアコーディオン の top 値に mainAccordionHeight を加味する。
スクロールイベント で各セクション要素の getBoundingClientRect().top をチェックして、上部付近にあるセクション名を activeSection に更新。
これにより、

メインアコーディオンが開閉でサイズ変わってもサブアコーディオンの位置がズレない。
いまどのセクションが画面上部にあるかがリアルタイムでわかる。
```
import React, { useEffect, useState, useRef } from "react";
import { Accordion, AccordionSummary, AccordionDetails } from "@mui/material";
import ExpandMoreIcon from "@mui/icons-material/ExpandMore";
import {
  Container,
  MainAccordionWrapper,
  SubAccordionWrapper,
  SectionBlock,
  SectionTitle,
  ScrollSpyBlock,
} from "./styled-components"; // 先ほどの styled-components ファイルを読み込む
import type { ResizeObserverEntry } from "resize-observer"; // 型補完用 (環境に応じインストール)

type SectionData = {
  title: string;
  description?: string;
  items: { id: number; name: string; detail: string }[];
};

type DataType = {
  [key: string]: SectionData;
};

interface Props {
  data: DataType; // JSONデータ
}

const MultiAccordionExample: React.FC<Props> = ({ data }) => {
  // メインアコーディオンの高さを保持
  const [mainAccordionHeight, setMainAccordionHeight] = useState(0);

  // 現在、画面上部付近にあるセクション名
  const [activeSection, setActiveSection] = useState<string>("");

  // メインアコーディオンを参照する ref
  const mainAccordionRef = useRef<HTMLDivElement | null>(null);

  // スクロール制御用
  const tickingRef = useRef(false);

  // =============================================
  // 1) メインアコーディオンのサイズ変化を検知して更新
  // =============================================
  useEffect(() => {
    if (!mainAccordionRef.current) return;

    const observer = new ResizeObserver((entries: ResizeObserverEntry[]) => {
      // contentRect.height から要素の高さを取得
      const { height } = entries[0].contentRect;
      setMainAccordionHeight(height);
    });

    observer.observe(mainAccordionRef.current);

    // コンポーネントのアンマウント時に監視を停止
    return () => {
      observer.disconnect();
    };
  }, []);

  // =============================================
  // 2) スクロール中に、いま表示中のセクションを判定
  // =============================================
  useEffect(() => {
    const handleScroll = () => {
      if (!tickingRef.current) {
        window.requestAnimationFrame(() => {
          // スクロール時に全セクションのブロックを取得
          const sectionBlocks = document.querySelectorAll(".scroll-section");

          let newActiveSection = activeSection;

          sectionBlocks.forEach((block) => {
            const rect = block.getBoundingClientRect();
            // 上端が 200px 以内に来た場合 (基準値は適宜調整)
            if (rect.top <= 200) {
              // data-section-title に格納した文字列をアクティブ表示
              const title = block.getAttribute("data-section-title") || "";
              newActiveSection = title;
            }
          });

          if (newActiveSection !== activeSection) {
            setActiveSection(newActiveSection);
          }

          tickingRef.current = false;
        });
        tickingRef.current = true;
      }
    };

    window.addEventListener("scroll", handleScroll);

    return () => {
      window.removeEventListener("scroll", handleScroll);
    };
  }, [activeSection]);

  // データのキー一覧 (セクションを識別するID)
  const sectionKeys = Object.keys(data);

  return (
    <Container>
      {/* ========================= */}
      {/* メインアコーディオン   */}
      {/* ========================= */}
      <MainAccordionWrapper ref={mainAccordionRef}>
        <Accordion>
          <AccordionSummary expandIcon={<ExpandMoreIcon />}>
            <strong>Main Accordion</strong>
          </AccordionSummary>
          <AccordionDetails>
            ここは画面上部に固定されるアコーディオンです。<br/>
            開閉することで高さが変わります。
          </AccordionDetails>
        </Accordion>
      </MainAccordionWrapper>

      {/* ========================= */}
      {/* サブアコーディオン      */}
      {/* ========================= */}
      <SubAccordionWrapper offsetTop={mainAccordionHeight}>
        <Accordion>
          <AccordionSummary expandIcon={<ExpandMoreIcon />}>
            <strong>Sub Accordion</strong>
          </AccordionSummary>
          <AccordionDetails>
            メインアコーディオンの高さだけ余白をとり、上部に固定して表示します。
          </AccordionDetails>
        </Accordion>
      </SubAccordionWrapper>

      {/* ========================= */}
      {/* セクション一覧表示       */}
      {/* ========================= */}
      {sectionKeys.map((key) => {
        const section = data[key];
        const isActive = activeSection === section.title;

        return (
          <SectionBlock
            key={key}
            className="scroll-section"
            data-section-title={section.title}
          >
            <SectionTitle style={{ color: isActive ? "red" : "inherit" }}>
              {section.title}
            </SectionTitle>
            <p>{section.description}</p>
            <ScrollSpyBlock>
              {/* セクション内の個別アイテムを羅列 */}
              {section.items.map((item) => (
                <div key={item.id}>
                  <b>{item.name}</b>: {item.detail}
                </div>
              ))}
            </ScrollSpyBlock>
          </SectionBlock>
        );
      })}
    </Container>
  );
};

export default MultiAccordionExample;

```

解説
メインアコーディオン (MainAccordionWrapper)

position: sticky; top: 0; としているため、ページをスクロールしても常に最上部に追従します。
ResizeObserver で 高さ変化 を監視し、mainAccordionHeight に反映します。
サブアコーディオン (SubAccordionWrapper)

こちらも position: sticky; ですが、top: mainAccordionHeight を指定することで、メインのアコーディオンが占有している高さ分だけ下に固定されます。
メインアコーディオンが開閉でサイズ変わると、 mainAccordionHeight が更新され、サブアコーディオンの位置も動的に変化します。
スクロール位置の追跡

useEffect 内で scroll イベントを監視。
requestAnimationFrame とフラグ (tickingRef) を使うことで、連続するスクロールイベントの処理を最適化。
各セクションブロックの getBoundingClientRect().top をチェックし、画面上部付近にあるセクション名を activeSection に保持。
表示中のセクションを反映させるため、タイトルの文字色を変える (isActive ? "red" : "inherit" など)。

まとめ
2つ以上の Sticky アコーディオンを重ねて表示する場合、上のアコーディオンの高さが変わると下の要素が見切れてしまうことがあります。
ResizeObserver で高さを監視し、サブ要素の top を更新 すれば、アコーディオンの開閉に動的に対応可能です。
また、スクロールによって画面上部に来ている セクション(タイトル) を判定したいときは、getBoundingClientRect() の .top 位置を確認しながら、状態管理 で追跡するのがおすすめです。
これで、React + MUI アコーディオン を使った複数要素の「追従表示 + セクション切り替え」が簡単に実現できます。
