---
title: <%*
  let userTitle = await tp.system.prompt("새 게시물의 제목을 입력하세요");
  if (!userTitle) {
    userTitle = "new-post";
  }

  let hyphenTitle = userTitle.replace(/\s+/g, "-");

  tR += hyphenTitle;
%>
date: <% tp.date.now("YYYY-MM-DD") %>
categories: 
tags:
toc: true
pin: true
sitemap :
  changefreq : daily
  priority : 1.0
image:
  path: 
  alt: 
---

<%*
  const newFileName = `${tp.date.now("YYYY-MM-DD")}-${hyphenTitle}`;
  tR += await tp.file.rename(newFileName);
%>


<script src="https://giscus.app/client.js"
        data-repo="YeaChan05/YeaChan05.github.io"
        data-repo-id="R_kgDONnXleQ"
        data-category="Announcements"
        data-category-id="DIC_kwDONnXlec4Cl2a5"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="ko"
        crossorigin="anonymous"
        async>
</script>