---
title: 多行文本缩略的实现
categories:
  - FE
abbrlink: ef1ff28e
date: 2023-01-05 09:22:35
---

目前大多数浏览器不支持多行文本缩略，因此需要做兼容处理。比较好的思路是在首次渲染后去读取每个文字的宽度，计算是否应该展示缩略效果。为了能够读取每个文字的宽度，需要使用 span 包裹每个文字。

{% note info %}

1. 在非缩略状态下，通过读取每个文字的宽度，计算是否应该缩略
2. 如果需要缩略，则使用一个 span 包裹前几行不需要缩略的文字，用另一个 span 包裹后几行需要缩略的文字
3. 如果不需要缩略，使用 span 包裹每一个文本
4. 如果文本更新了
   3.1 如果当前处于缩略状态，则先重置为非缩略状态，然后在跳转到步骤 1
   3.2 如果当前处于非缩略状态，则直接跳转到步骤 1
   {% endnote %}

React 的示例代码如下所示：

```ts
import React, { useState, useRef, useEffect } from 'react';

export interface EllipsisMultipleLineTextProps {
  text?: string;
  maxLine?: number;
  className?: string;
  onEllipsis?: (isEllipsis: boolean) => void;
}

export default function EllipsisMultipleLineText(
  props: EllipsisMultipleLineTextProps
) {
  const { text, maxLine = 1, className, onEllipsis } = props;

  const [isEllipsis, setIsEllipsis] = useState(false);
  const [ellipsisLineFirstTextIndex, setEllipsisLineFirstTextIndex] =
    useState(0);
  const [rerenderAfterReset, setRenderAfterReset] = useState(0);

  const divRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (isEllipsis) {
      setIsEllipsis(false);
      setRenderAfterReset((pre) => pre + 1);
      return;
    }

    if (divRef.current && text) {
      const { offsetWidth } = divRef.current;

      const headIndexList: number[] = [0];
      let cumulativeWidth = 0;
      const children = divRef.current.children as never as HTMLSpanElement[];

      for (let i = 0; i < children.length; i++) {
        if (cumulativeWidth + children[i].offsetWidth <= offsetWidth) {
          cumulativeWidth += children[i].offsetWidth;
        } else if (cumulativeWidth + children[i].offsetWidth > offsetWidth) {
          cumulativeWidth = 0;
          headIndexList.push(i);
        }
      }

      if (headIndexList.length > maxLine) {
        setIsEllipsis(true);
        setEllipsisLineFirstTextIndex(headIndexList[maxLine - 1]);
      }
    }
  }, [text, rerenderAfterReset]);

  useEffect(() => {
    onEllipsis?.(isEllipsis);
  }, [isEllipsis]);

  const renderText = () => {
    if (!text) {
      return null;
    }

    if (!isEllipsis) {
      return text?.split('').map((item) => <span>{item}</span>);
    }

    return (
      <>
        <span>{text.slice(0, ellipsisLineFirstTextIndex)}</span>
        <span
          style={{
            display: 'inline-block',
            width: '100%',
            overflow: 'hidden',
            textOverflow: 'ellipsis',
            whiteSpace: 'nowrap',
          }}
        >
          {text.slice(ellipsisLineFirstTextIndex)}
        </span>
      </>
    );
  };

  return (
    <div className={className} ref={divRef}>
      {renderText()}
    </div>
  );
}
```
