---
title: RAG
categories: ["RAG", "AI"]
tags: ["RAG", "ai","chatbot","Automated"]
date: 2026-04-22
draft: false
showauthor: true
authors:
  - Bubberr
---
# Automated RAG Workflow for Keeping My Dify Chatbot in Sync with My Portfolio

## Overview
In this post, I’ll walk through how I built an automated workflow that ensures my RAG (Retrieval-Augmented Generation) chatbot in Dify always reflects the latest content from my portfolio.

The goal was simple:
> Whenever I update my portfolio, my chatbot should automatically “know” about it.

To achieve this, I created a Java-based pipeline that:
1. Reads my sitemap  
2. Extracts all portfolio URLs  
3. Converts each page into clean Markdown  
4. Uploads the content into a Dify dataset  

---

##  Architecture

---

## ⚙️ Technologies Used

- Java
- OkHttp (HTTP client)
- Jsoup (XML parsing)
- Jina AI Reader API (HTML → Markdown)
- Dify API (dataset ingestion)

---

## 🔄 Step 1: Fetch URLs from Sitemap

I start by fetching my sitemap and extracting all `<loc>` elements using Jsoup.

```java
public static List<String> fetchSitemapUrls(String sitemapUrl) throws IOException {
    Request request = new Request.Builder().url(sitemapUrl).build();
    Response response = client.newCall(request).execute();

    String xml = response.body().string();

    Document doc = Jsoup.parse(xml, "", org.jsoup.parser.Parser.xmlParser());
    Elements locElements = doc.select("loc");

    List<String> urls = new ArrayList<>();
    locElements.forEach(e -> urls.add(e.text()));

    return urls;
}

public static String fetchMarkdownFromJina(String url) throws IOException {
    String jinaUrl = JINA_API + url;

    Request request = new Request.Builder()
            .url(jinaUrl)
            .header("Accept", "text/plain")
            .build();

    Response response = client.newCall(request).execute();

    if (!response.isSuccessful()) return null;

    return response.body().string();
}
# Automated RAG Bot with Dify – Always in Sync with My Portfolio

> **TL;DR:** I built a Java program that automatically reads all pages from my portfolio sitemap, converts them to Markdown via Jina AI, and uploads them to a Dify knowledge base — so my RAG chatbot always reflects the latest content on my site.

---

## Background

I have a portfolio hosted on GitHub Pages and an AI chatbot built with [Dify](https://dify.ai) that visitors can use to ask questions about my work. The problem was: **every time I updated my portfolio, the chatbot was still answering based on outdated content.**

The solution? An automated workflow that syncs content from my portfolio directly into Dify's RAG knowledge base — with no manual intervention.

---

## Architecture & Flow

The workflow consists of three steps:

```
Portfolio (sitemap.xml)
        │
        ▼
  [1] Fetch all page URLs from sitemap
        │
        ▼
  [2] Convert each page to Markdown via Jina AI
        │
        ▼
  [3] Upload Markdown files to Dify dataset via API
```

Everything is handled by a single Java program: `SitemapToDify.java`.

---

## Implementation

### Step 1 – Fetch URLs from the Sitemap

I use `OkHttp` to fetch `sitemap.xml` and `Jsoup` to parse the XML and extract all `<loc>` tags:

```java
public static List<String> fetchSitemapUrls(String sitemapUrl) throws IOException {
    Request request = new Request.Builder().url(sitemapUrl).build();
    Response response = client.newCall(request).execute();

    String xml = response.body().string();
    Document doc = Jsoup.parse(xml, "", org.jsoup.parser.Parser.xmlParser());
    Elements locElements = doc.select("loc");

    List<String> urls = new ArrayList<>();
    locElements.forEach(e -> urls.add(e.text()));
    return urls;
}
```

The result is a complete list of every page URL on my portfolio — automatically, with no hardcoding.

---

### Step 2 – Convert Pages to Markdown via Jina AI

[Jina AI's Reader API](https://jina.ai/reader/) takes a URL and returns the page content as clean Markdown — perfect for RAG, since it strips away all the noise: HTML tags, navigation, footers, and ads:

```java
public static String fetchMarkdownFromJina(String url) throws IOException {
    String jinaUrl = "https://r.jina.ai/" + url;

    Request request = new Request.Builder()
            .url(jinaUrl)
            .header("Accept", "text/plain")
            .build();

    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) return null;

    return response.body().string();
}
```

Just prefix any URL with `https://r.jina.ai/` and Jina returns clean Markdown. Simple and effective.

---

### Step 3 – Upload to Dify Dataset

Each Markdown file is uploaded as a document to my Dify knowledge base via their REST API. I use `multipart/form-data` with two parts: the file itself and a JSON configuration string:

```java
public static void uploadToDify(String url, String markdown) throws IOException {
    String fileName = url.replaceAll("[^a-zA-Z0-9]", "_") + ".md";

    String dataJson = new JSONObject()
            .put("indexing_technique", "high_quality")
            .put("doc_form", "text_model")
            .put("doc_language", "English")
            .put("process_rule", new JSONObject().put("mode", "automatic"))
            .toString();

    RequestBody requestBody = new MultipartBody.Builder()
            .setType(MultipartBody.FORM)
            .addFormDataPart("file", fileName,
                    RequestBody.create(markdown, MediaType.parse("text/markdown")))
            .addFormDataPart("data", dataJson)
            .build();

    Request request = new Request.Builder()
            .url(DIFY_API)
            .post(requestBody)
            .addHeader("Authorization", "Bearer " + DIFY_API_KEY)
            .build();

    client.newCall(request).execute();
}
```

One important detail: the `data` field **must** be a serialized JSON string, not a nested object — this caught me off guard and took a while to figure out.

---

### Main Loop

Everything is tied together in `main()`:

```java
public static void main(String[] args) throws Exception {
    List<String> urls = fetchSitemapUrls(SITEMAP_URL);

    for (String url : urls) {
        System.out.println("Processing: " + url);
        String markdown = fetchMarkdownFromJina(url);
        if (markdown != null && !markdown.isEmpty()) {
            uploadToDify(url, markdown);
        }
        Thread.sleep(1000); // avoid rate limits
    }
}
```

I add a one-second delay between each page to avoid hitting rate limits on Jina's API.

---

## Technologies Used

| Technology | Purpose |
|---|---|
| **Java** | Programming language for the workflow |
| **OkHttp** | HTTP client for API calls |
| **Jsoup** | XML/HTML parsing of the sitemap |
| **Jina AI Reader** | Converting web pages to Markdown |
| **Dify** | RAG platform and chatbot engine |
| **GitHub Pages** | Portfolio hosting (Hugo) |

---

## What I Learned

**Jina AI Reader is underrated.** Converting an entire web page to clean Markdown with a single API call is extremely useful for RAG pipelines — no scraping logic, no HTML parsing, just content.

**Dify's API is well-documented, but detail-oriented.** The `data` field in the multipart request must be a serialized JSON string, not an object. Easy to overlook, but critical for the upload to succeed.

**Automation pays off quickly.** I can now run the script after every portfolio update — or set it up as a GitHub Action that triggers on every push — and my chatbot is always up to date.

---

## Next Steps

- [ ] Set up the workflow as a **GitHub Action** that runs automatically on every push to the portfolio repo
- [ ] Add **deletion of old documents** in Dify before re-uploading, to prevent duplicate entries from accumulating
- [ ] Add **language detection** instead of hardcoding `"English"`

---

*The code is written as a standalone Java program. If you have questions about the implementation, feel free to reach out.*