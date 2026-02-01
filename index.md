---
layout: home
title: 技术笔记
description: 记录 AI 学习与开发实践
---

<div class="hero">
  <h1>Hello，我是 <span class="highlight">开发者</span></h1>
  <p class="subtitle">专注 AI Agent 与大语言模型应用开发</p>
</div>

<div class="intro">
  <p>这里是我的技术博客，主要分享：</p>
  <ul>
    <li>🤖 AI Agent</li>
    <li>📝 学习笔记与思考</li>
  </ul>
</div>

<div class="featured-section">
  <h2>🎯 重点专题</h2>
  
  <div class="featured-grid">
    <a href="/ai-agent-learning/" class="featured-card">
      <div class="icon">🤖</div>
      <h3>AI Agent 学习</h3>
      <p>从基础到实践的 AI Agent 开发指南</p>
      <span class="count">{{ site.categories.ai-agent | size }} 篇文章</span>
    </a>
  </div>
</div>

<div class="recent-posts">
  <div class="section-header">
    <h2>📝 最新文章</h2>
    <a href="/archive/" class="view-all">查看全部 →</a>
  </div>
  
  {% for post in site.posts limit:4 %}
  <article class="post-card">
    <div class="post-meta">
      <time>{{ post.date | date: "%Y.%m.%d" }}</time>
      <span class="category">
        {% for category in post.categories limit:1 %}
          {{ category }}
        {% endfor %}
      </span>
    </div>
    <h3>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </h3>
    <p class="excerpt">{{ post.excerpt | strip_html | truncate: 120 }}</p>
    <div class="post-footer">
      <div class="tags">
        {% for tag in post.tags limit:2 %}
          <span class="tag">{{ tag }}</span>
        {% endfor %}
      </div>
      <a href="{{ post.url }}" class="read-more">阅读</a>
    </div>
  </article>
  {% endfor %}
</div>

<div class="ai-agent-preview">
  <h2>🤖 AI Agent 学习笔记</h2>
  <div class="ai-posts">
    {% assign ai_posts = site.categories.ai-agent %}
    {% for post in ai_posts limit:4 %}
      <a href="{{ post.url }}" class="ai-post-item">
        <span class="ai-post-title">{{ post.title }}</span>
        <span class="ai-post-date">{{ post.date | date: "%m.%d" }}</span>
      </a>
    {% endfor %}
  </div>
  <a href="/ai-agent-learning/" class="ai-more-link">查看所有 AI 文章 →</a>
</div>

<div class="contact-section">
  <h2>📬 Contact Me</h2>
  <div class="contact-links">
    <a href="sijinwang00@gmail.com" class="contact-link">
      <span class="contact-icon">📧</span>
      <span>Email</span>
    </a>
    <a href="https://github.com/ksyszk" class="contact-link" target="_blank">
      <span class="contact-icon">🐙</span>
      <span>GitHub</span>
    </a>
    <a href="/rss.xml" class="contact-link">
      <span class="contact-icon">📡</span>
      <span>RSS</span>
    </a>
  </div>
</div>