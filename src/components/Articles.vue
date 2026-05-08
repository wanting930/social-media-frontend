<script setup>
</script>

<template>
  <div class="articles-container">
    <div class="articles-card">
      <div class="articles-header">
        <h1>瀏覽文章</h1>
        <p>歡迎回來！以下是最新文章列表。</p>
      </div>

      <div class="articles-list">
        <article class="article-item">
          <h2>第一篇文章標題</h2>
          <p>這裡是文章摘要，用來展示文章列表頁面的樣式。</p>
        </article>
        <article class="article-item">
          <h2>第二篇文章標題</h2>
          <p>你可以把這裡替換成從後端撈到的文章內容。</p>
        </article>
        <article class="article-item">
          <h2>第三篇文章標題</h2>
          <p>這個頁面只是示範登入成功後的跳轉新頁面。</p>
        </article>
      </div>
    </div>
  </div>
</template>

<style scoped>
.articles-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background: linear-gradient(135deg, #f5f7ff 0%, #dfe7fd 100%);
  padding: 20px;
}

.articles-card {
  width: 100%;
  max-width: 900px;
  background: white;
  border-radius: 18px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.12);
  padding: 36px;
}

.articles-header {
  margin-bottom: 24px;
}

.articles-header h1 {
  margin: 0 0 10px 0;
  font-size: 32px;
  color: #222;
}

.articles-header p {
  margin: 0;
  color: #555;
  font-size: 16px;
}

.articles-list {
  display: grid;
  gap: 18px;
}

.article-item {
  padding: 22px;
  background: #f7f9ff;
  border-radius: 14px;
  border: 1px solid #e3e9ff;
}

.article-item h2 {
  margin: 0 0 10px 0;
  font-size: 20px;
  color: #1f2b45;
}

.article-item p {
  margin: 0;
  color: #555;
  line-height: 1.6;
}

@media (max-width: 640px) {
  .articles-card {
    padding: 24px;
  }

  .articles-header h1 {
    font-size: 26px;
  }
}
</style>
