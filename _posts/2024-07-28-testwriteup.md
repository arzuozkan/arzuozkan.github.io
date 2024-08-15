---
layout: post
title: "Remote Markdown Example"
---

# Welcome

**Hello world**, this is my first Jekyll blog post.

I hope you like it!

Aşağıda, uzak bir kaynaktan içerik çekip gösteren JavaScript örneğini bulabilirsiniz.

<div id="remote-content">
    <!-- İçerik buraya yüklenecek -->
</div>

<script>
    fetch('https://raw.githubusercontent.com/arzuozkan/writeup_ctf_notes/master/devops-hacking/frankandherbytryagain.md')
        .then(response => response.text())
        .then(data => {
            document.getElementById('remote-content').innerText = data;
        })
        .catch(error => console.error('Fetch işlemi başarısız oldu:', error));
</script
