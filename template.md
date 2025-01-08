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
---

<%*
  const newFileName = `${tp.date.now("YYYY-MM-DD")}-${hyphenTitle}`;
  tR += await tp.file.rename(newFileName);
%>
