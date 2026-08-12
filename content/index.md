---
title: "Factsheet Download Automation"
---

<style>
  .home-hero-section {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 3rem;
    margin-top: 3rem;
    margin-bottom: 4rem;
  }
  .home-hero-content {
    flex: 1;
  }
  .home-hero-content h2 {
    margin-top: 0;
    font-size: 1.75rem;
    font-weight: 700;
    line-height: 1.3;
  }
  .home-hero-image {
    flex: 1;
    display: flex;
    justify-content: center;
  }
  .home-hero-image img {
    max-width: 100%;
    height: auto;
  }
  .home-about-card {
    display: flex;
    align-items: center;
    background: #17181e; /* Dark card background */
    border-radius: 12px;
    padding: 2.5rem;
    gap: 3rem;
    margin-top: 2rem;
    box-shadow: 0 10px 30px rgba(0,0,0,0.2);
  }
  .home-about-icon {
    flex-shrink: 0;
  }
  .home-about-icon img {
    width: 160px;
    height: auto;
  }
  .home-about-content {
    flex: 1;
    border-left: 1px solid rgba(255, 255, 255, 0.1);
    padding-left: 2.5rem;
  }
  .home-about-content h3 {
    margin-top: 0;
    font-size: 1.5rem;
    font-weight: 600;
  }
  
  /* Media query for mobile */
  @media (max-width: 768px) {
    .home-hero-section {
      flex-direction: column;
      gap: 2rem;
    }
    .home-about-card {
      flex-direction: column;
      text-align: center;
      padding: 1.5rem;
      gap: 1.5rem;
    }
    .home-about-content {
      border-left: none;
      border-top: 1px solid rgba(255, 255, 255, 0.1);
      padding-left: 0;
      padding-top: 1.5rem;
    }
  }
</style>

<div class="home-hero-section">
  <div class="home-hero-content">
    <h2>Automated Mutual Fund Factsheet Collection</h2>
    <p>A Python-based automation system designed to retrieve and download the latest mutual fund factsheets from Asset Management Company (AMC) websites.</p>
    <p>The system reduces manual effort by automating website navigation, factsheet identification, URL extraction, and document downloading.</p>
  </div>
  <div class="home-hero-image">

![[hero.png|Automation Hero Image]]

  </div>
</div>

<div class="home-about-card">
  <div class="home-about-icon">

![[about.png|About Icon]]

  </div>
  <div class="home-about-content">
    <h3>About This Documentation</h3>
    <p>This documentation explains the architecture, setup, workflow, and maintenance of the Factsheet Download Automation system.</p>
    <p>It is intended to help developers understand the project and make future maintenance and enhancements easier.</p>
  </div>
</div>