---
title: Gatsby에 category 추가하는법
date: "2022-03-15"
desc: "Gatsby에 Category 적용 방법에 대해 적었습니다."
alt: "Gatsby에 category 추가하는법"
category: "JavaScript"
tags: [ "Gatsby", "Category", "React", "Javascript" ]
thumbnail: ./images/Gatsby_logo.png
---

`※ Gatsby로 블로그를 만들었단 전제하에 작성된 글 입니다.`  

블로그 개발 일지를 작성하면서 Gatsby에 카테고리 적용 방법에 대해 같이 적을려고 했었으나 퇴근 시간대에 가까워져 다음에 쓰기로 하다가,
오늘 쓰게 되었다.

# 참고 사이트

Gatsby에 카테고리 적용하는법에 대해서는 여러 사이트를 참고하면서 만들었다.

 - https://soopdop.github.io/2020/11/27/add-categories-to-gatsby/
 - https://usgnuscodenote.tistory.com/entry/gatsby-블로그-카테고리-만들기-1-gatsby-blog-add-category-1
 - https://ichi.pro/ko/gatsby-beullogeue-taegeuleul-chugahaneun-bangbeob-233776920738588
 
# Frontmatter에 categories 추가

markdown 문서 가장 위에 있는 메타데이터를 frontmatter 이라 한다. (graphQL로 데이터를 가져올 때 위에 정보들이 frontmatter 이란 object 안에 들어있다.)
여기에 categories이란 이름(난 category로 지었다.)에 값은 쓰고자 하는 사람이 원하는데로 적어준다.

```markdown
---

categories: "Javascript"

---
```

나의 경우에는 문자열 형태로 하고 싶어 문자열 형태로 저장했고 `/` 로 구분을 주었다.
(`/` 이외의 딴걸로 구분지어줘도 된다. / 배열 형태로 category를 구분하여 작성해본 적은 없어서 이쪽은 잘 모르겠다.)

# Category Component 생성

`src\components\` 경로에 category.js (typescript로 만들어도 무관하다.)를 생성해준다.

```javascript

import {Link} from "gatsby";
import React from "react";
import {kebabCase} from "./util";
import { GoTriangleDown } from "react-icons/go";

const MenuCategory = ({categories, thisCategory}) => {
    if (categories === undefined) {
        return (
            <ul className="categories">
            </ul>
        );
    }
    // .md파일의 category = "category/subCategory" 형태의 데이터들을 배열 형태로 가져와 정제
    // ex) const refineCategories = [{ fieldValue: "category/subCategory, totalCount: 2 }]
    // => [ { fieldValue: "category", totalCount:2, subCategory: [ { fieldValue: "subCategory", totalCount: 2, subCategory:[] } ]  } ]
    // 으로 변환되게 만들었음.
    // 아직 변수명을 잘 못지어서 고심 끝에 짓긴 하였지만 이상할 수 있음
    const refineCategoies = categories.reduce((acc, {fieldValue, totalCount}, idx) => {
        const fFirstObjIdx = (value) => acc.findIndex((data, dIdx) => data.fieldValue !== undefined && data.fieldValue === value);
        const categoryArr = fieldValue.split("/");

        if (fFirstObjIdx(categoryArr[0]) === -1) {
            acc.push({
                fieldValue: categoryArr[0],
                totalCount: (categoryArr.length <= 1 ? totalCount : 0),
                subCategory: [],
                categoryPath: categoryArr[0]
            });
        }

        let mainTarget = acc[fFirstObjIdx(categoryArr[0])];
        let subTarget = mainTarget;
        categoryArr.forEach((category, dIdx) => {
            // 0번째는 이미 mainTarget에 있으므로 0번째는 넘김.
            if (dIdx === 0) return true;

            let subTargetIdx = subTarget?.subCategory.findIndex((d) => d.fieldValue !== undefined && d.fieldValue === category) || -1;

            // subTarget에 해당 카테고리가 없으면 새로 생성
            // 해당 카테고리가 있으면 totalCount 1씩 증가
            if (subTargetIdx === -1) {
                const categoryPath = [...categoryArr].slice(0, dIdx + 1).join("/");
                subTarget.subCategory.push({fieldValue: category, totalCount, subCategory: [], categoryPath});

                subTargetIdx = 0;
            } else {
                subTarget.totalCount += 1;
            }

            mainTarget.totalCount += 1;

            // subTarget을 해당 카테고리의 서브 카테고리 값으로 변경
            subTarget = subTarget.subCategory[subTargetIdx];
        });

        return acc;
    }, []);

    const toggleSubCategory = e => {
        e.stopPropagation()
        const {target} = e;

        if(target.tagName === "SPAN") {
            target.parentNode.classList.toggle("testActive");
        } else {
            target.classList.toggle("testActive");
        }
    }

    const subCategoryTag = (subCategoryArr = [], depth = 1) => {
        if (subCategoryArr.length === 0) return "";

        return (
            <ul className="subCategory" data-depth={depth}>
                {subCategoryArr.map(({fieldValue, totalCount, subCategory, categoryPath}, cIdx) =>
                    (<li
                        key={fieldValue}
                        className={fieldValue === thisCategory ? "active" : ""}
                        onClick={(e) => toggleSubCategory(e)}>
                        {subCategory.length !== 0 ? <span>{fieldValue} ({totalCount}) <GoTriangleDown /></span> :
                            <Link to={"/category/" + kebabCase(categoryPath)}>{fieldValue} ({totalCount})</Link>}
                        {subCategoryTag(subCategory, depth + 1)}
                    </li>))}
            </ul>);
    };

    return (
        <ul className="categories">
            {refineCategoies.map(({fieldValue, totalCount, subCategory}) => {
                return (
                    <li key={fieldValue} className={fieldValue === thisCategory ? "active" : ""}
                        onClick={(e) => toggleSubCategory(e)}>
                        {subCategory.length !== 0 ? <span>{fieldValue} ({totalCount}) <GoTriangleDown /></span> :
                            <Link to={`/category/${kebabCase(fieldValue)}`}>{fieldValue}({totalCount})</Link>}
                        {subCategoryTag(subCategory)}
                    </li>
                );
            })}
        </ul>
    );
};

export default MenuCategory;

```

markdown의 메타데이터의 categories(category) 부분을 문자열로 한 경우 어느정도 변환을 해 주어야 한다.
보통 graphQL을 통해 categories 데이터를 가져올 때 (아래부분에 추가 설명 되어있음) 나와 같이 문자열로 한 경우엔
```javascript
[
    { fieldValue: "Javascript/React", totalCount: 1 },
    { fieldValue: "Javascript/React/Basic", totalCount: 1},
    { fieldValue: "Javascript/React/Normal", totalCount: 1 },
    { fieldValue: "Javascript/Typescript", totalCount: 1 }
]
```

등으로 가져와질 것이다. (한 markdown파일 에서만이 아닌 모든 markdown파일 에서 메타데이터를 가져온다.)
ul>li 의 형태로 카테고리를 만들고자 한다면 기존에 가져온 형태로는 만들기 어렵단 생각이 들어 변환을 주었다.
나 같은 경우는 좀 더 쉽게 만들기 위해서

```javascript
[
    { 
        fieldValue: "Javascript", 
        totalCount:4,
        categoryPath: "Javascript",
        subCategory: [
            { 
                fieldValue: "React",
                totalCount: 3,
                categoryPath: "Javascript/React",
                subCategory: [
                    { fieldValue: "Basic", totalCount: 1, categoryPath: "Javascript/React/Basic"  ,subCategory: [] },
                    { fieldValue: "Normal", totalCount: 1, categoryPath: "Javascript/React/Normal"  ,subCategory: [] }
                ]
            },
            {
                fieldValue: "Typescript",
                totalCount: 1,
                categoryPath: "Javascript/Typescript",
                subCategory: []
            }
        ] 
    }
]
```
위와 같이 변환 해주었고, 변환해준 코드가 전체 코드 부분중 `refineCategoies` 란 변수에 담겨있다. ( 다른 방법이 있을수도 있지만 나는 가장먼저 이 방법이 떠올라 이런 형식으로 만들었다. )




나의 경우에는 메뉴 아이콘 클릭시 숨겨진 메뉴가 나타나 카테고리가 보이는 식으로 만들었다. 글 왼쪽에 카테고리를 추가 하고 싶은 경우에는
`src\components\layout.js`에서
