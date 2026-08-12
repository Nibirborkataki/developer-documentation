---
title: "Factsheet Automation"
---

<style>
  .fs-hero-container {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 3rem;
    margin-top: 2rem;
    margin-bottom: 3rem;
  }
  .fs-hero-text {
    flex: 1;
  }
  .fs-hero-title-wrap {
    display: flex;
    align-items: center;
    gap: 1rem;
    margin-bottom: 1rem;
  }
  .fs-hero-title-wrap h3 {
    margin: 0;
    font-size: 1.5rem;
    font-weight: 700;
  }
  .fs-hero-icon {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 44px;
    height: 44px;
    background: rgba(123, 97, 255, 0.1);
    color: #7b61ff;
    border-radius: 8px;
    flex-shrink: 0;
  }
  .fs-hero-text p {
    color: #a0a0a0;
    line-height: 1.6;
    margin: 0;
  }
  .fs-hero-image {
    flex: 1;
    display: flex;
    justify-content: flex-end;
  }
  .fs-hero-image img {
    max-width: 100%;
    height: auto;
  }
  .fs-divider {
    height: 1px;
    background: rgba(255, 255, 255, 0.1);
    margin: 3rem 0;
    border: none;
  }
  .fs-chapters-header {
    display: flex;
    align-items: center;
    gap: 1rem;
    margin-bottom: 0.5rem;
  }
  .fs-chapters-header h3 {
    margin: 0;
    font-size: 1.5rem;
    font-weight: 700;
  }
  .fs-chapters-desc {
    color: #a0a0a0;
    margin-bottom: 2rem;
  }
  .fs-cards-container {
    display: flex;
    flex-direction: column;
    gap: 1.25rem;
  }
  .fs-card {
    display: flex;
    align-items: center;
    background: #17181e;
    border: 1px solid rgba(255, 255, 255, 0.05);
    border-radius: 12px;
    padding: 1.5rem 2rem;
    text-decoration: none !important;
    transition: all 0.2s ease;
    gap: 2rem;
  }
  .fs-card:hover {
    background: #1c1d24;
    border-color: rgba(123, 97, 255, 0.3);
    transform: translateY(-2px);
  }
  .fs-card-num {
    font-size: 2.25rem;
    font-weight: 700;
    color: #7b61ff;
    min-width: 50px;
    text-align: center;
  }
  .fs-card-icon {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 60px;
    height: 60px;
    background: rgba(123, 97, 255, 0.05);
    border: 1px solid rgba(123, 97, 255, 0.2);
    border-radius: 12px;
    color: #7b61ff;
    flex-shrink: 0;
  }
  .fs-card-content {
    flex: 1;
  }
  .fs-card-content h4 {
    margin: 0;
    color: #ffffff;
    font-size: 1.1rem;
    font-weight: 700;
  }
  .fs-card-content .subtitle {
    display: block;
    color: #7b61ff;
    font-size: 0.9rem;
    margin: 0.2rem 0 0.4rem 0;
  }
  .fs-card-content p {
    margin: 0;
    color: #a0a0a0;
    font-size: 0.9rem;
    line-height: 1.5;
  }
  .fs-card-arrow {
    color: #7b61ff;
    opacity: 0.7;
    transition: transform 0.2s;
  }
  .fs-card:hover .fs-card-arrow {
    transform: translateX(4px);
    opacity: 1;
  }
  
  /* On mobile */
  @media (max-width: 768px) {
    .fs-hero-container {
      flex-direction: column;
      gap: 2rem;
    }
    .fs-card {
      flex-direction: column;
      text-align: center;
      gap: 1rem;
      padding: 1.5rem;
    }
    .fs-card-arrow {
      display: none;
    }
  }

  /* Hide default Quartz folder list at the bottom */
  .page .article-content > ul,
  .page .article-content .folder-outer,
  .page .article-content .nav-folder {
    display: none !important;
  }
</style>

<div class="fs-hero-container">
  <div class="fs-hero-text">
    <div class="fs-hero-title-wrap">
      <div class="fs-hero-icon">
        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="16" y1="13" x2="8" y2="13"></line><line x1="16" y1="17" x2="8" y2="17"></line><polyline points="10 9 9 9 8 9"></polyline></svg>
      </div>
      <h3>Documentation</h3>
    </div>
    <p>A complete guide to the Factsheet Download Automation system, covering its architecture, setup, directory structure, execution workflow, and maintenance.</p>
  </div>
  <div class="fs-hero-image">

![[factsheet.png|Documentation Image]]

  </div>
</div>

<hr class="fs-divider" />

<div class="fs-chapters-header">
  <div class="fs-hero-icon" style="background: transparent; color: #a0a0a0; padding: 0;">
    <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polygon points="12 2 2 7 12 12 22 7 12 2"></polygon><polyline points="2 17 12 22 22 17"></polyline><polyline points="2 12 12 17 22 12"></polyline></svg>
  </div>
  <h3>Documentation Chapters</h3>
</div>
<p class="fs-chapters-desc">Select a chapter below to explore the documentation.</p>

