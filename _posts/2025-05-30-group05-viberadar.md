---
layout: post
title: "Your Web App Is Live - Now What? How We Got Our First Users"
author: Michel Philippin & Leo Schöchli
---

After weeks (or months) of hard work building your web app, shipping it live can feel like the finish line. But in reality, **it's just the start**. This post outlines what we did *after* we launched [VibeRadar.io](https://viberadar.io) - our interactive globe-based app that maps the emotional "vibe" of cities using Spotify playlists and YouTube street footage.

![Screenshot of VibeRadar](/assets/img/2025-05-30-group05-viberadar-screenshot.png)

We’ll walk through adding analytics (without the need for a cookie banner), getting those precious first users, and learning from the data that came in.

## Step 1: Add analytics - but do it right

The first thing we needed was **visibility**: how many users do we have? What cities are they exploring? How long are they staying? Are they even clicking around?

We chose to use [PostHog](https://posthog.com), an open-source analytics platform with powerful event tracking. It's easy to set up. (You can also self-host if you prefer.)

### The immediate user turn-off: The Cookie Banner

Like most analytics tools, PostHog uses cookies to track users across sessions.
Try going to the first website which comes to your mind in Incognito mode. You’ll likely see a **cookie banner** asking for consent to track you. It’s annoying, right?

![Cookie Banners](/assets/img/2025-05-30-group05-viberadar-cookie-banners.png)

**We hate cookie banners too.** But here’s why they exist:

Cookie (and tracking) banners are required under European laws like the ePrivacy Directive ([Directive 2002/58/EC](https://eur-lex.europa.eu/eli/dir/2002/58/oj/eng), Article 5(3)) and the GDPR ([Regulation (EU) 2016/679](https://eur-lex.europa.eu/eli/reg/2016/679/oj/eng)). These laws say that you must get consent before placing non-essential cookies or using technologies that access personal data—such as IP addresses, device IDs, or behavioral profiles. (There are similar laws in other regions, like California’s CCPA.)

But here’s the good news:

> If you don’t track personal data, you often don’t need a banner at all.

If you want to read more about the psychology of cookie banners, check out this blog post by Nico Ebert: [The Psychology of Cookie Banners from a Data Privacy Perspective](https://blog.zhaw.ch/datascience/the-psychology-of-cookie-banners-from-a-data-privacy-perspective/).

### So how did we avoid cookie banners?

We configured PostHog to run in a **consentless, privacy-friendly** way.

That means:
- No cookies and no local storage (no tracking across sessions)
- No storing of personal data, i.e.:
  - No IP addresses
  - No city-level geolocation

The setup is simple. Here’s how we initialised PostHog in our app to ensure there are no cookies or local storage used to track users across sessions:

```js
posthog.init(POSTHOG_KEY, {
  persistence: "memory" // Default is "localStorage+cookie"
});
```

The business logic of our app **does not include any user data**. With help from our legal advisor (ChatGPT), we came to the conclusion that by additionally excluding IP addresses and city-level geolocation, we can run PostHog in a way that does not require a consent banner.

We set up a [Property Filter Transformation](https://posthog.com/tutorials/property-filter) in PostHog to remove `$ip`, `$geoip_latitude`, `$geoip_longitude` and `$geoip_city_name` from all events. This way, we only track usage data which cannot be used to identify individual users.

### We are not lawyers

We are no legal experts, and this is not legal advice. We tried our best to maximise user privacy while maintaining a good user experience, still getting valuable insights and adhering to the respective privacy laws.

As you can see these are a lot of interests to balance: **legal compliance, user privacy, user experience and product insights.**
The legal texts are complex and its interpretations vary. In case you consider using a similar setup for a large-scale app, please consult an expert.

Also, if you are an expert in this field yourself, we would really like to hear your thoughts on our approach.

## Step 2: Create a privacy policy

Even if you're not using cookies or storing personal information, you still need a privacy policy in many jurisdictions. It is also a good practice to be **transparent** to your users about what data you collect and why.

We created a lightweight but clear [Privacy Policy](https://viberadar.io/privacy-policy) for VibeRadar that explains:

* What data we collect (anonymous usage stats)
* Why we collect it (product improvement)
* How to contact us

Make it easy to find and easy to read.

## Step 3: Share Your App With the World - finally!

Once analytics and the legal side were in place, we started our attempt to get users. Here’s how we did it:

We shared the link to VibeRadar on various platforms, focusing on communities that appreciate new tech and data visualization. Here’s a breakdown of our initial traffic sources:

| Platform       | Visitors |
| -------------- | -------- |
| Direct link    | 47       |
| Hacker News    | 8        |
| X (Twitter)    | 2        |
| Google Search  | 2        |
| Reddit         | 1        |
| Bing           | 1        |

### Key Takeaways

* **Direct traffic** was dominant - likely from private sharing (friends, DMs, small groups).
* **Hacker News** sent real users who explored more deeply - good engagement.
* **Reddit/Twitter** performed poorly, but this could change with better targeting or phrasing.
* **Search traffic** was minimal, but we didn't optimize for search over internet yet.

We learned that getting users is a mix of **finding the right communities, timing and luck**. We will keep trying to find communities which might be interested in our project and sharing it with them. The key is to keep iterating and continuously getting feedback.

## Step 4: Measure What Matters

As you are reading this blog, VibeRadar is still live and (hopefully) growing. These are our current live stats:

<style>
  .metrics-container {
    display: flex;
    gap: 1.5rem;
    justify-content: space-between;
    padding: 1rem;
    flex-wrap: wrap;
  }

  .metric-box {
    flex: 1;
    min-width: 200px;
    padding: 1.5rem;
    background: #ffffff;
    border-radius: 16px;
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.05);
    text-align: center;
    transition: transform 0.2s ease;
  }

  .metric-box:hover {
    transform: translateY(-4px);
  }

  .metric-box h3 {
    font-size: 1rem;
    font-weight: 600;
    color: #333;
    margin-bottom: 0.5rem;
  }

  .metric-box p {
    font-size: 1.5rem;
    font-weight: 700;
    color: #111;
    margin: 0;
  }
</style>

<div class="metrics-container">
  <div class="metric-box">
    <h3>👥 This Week's Users</h3>
    <p id="this-weeks-users">Loading...</p>
  </div>

  <div class="metric-box">
    <h3>⏱️ Avg. Session Duration</h3>
    <p id="session-duration">Loading...</p>
  </div>

  <div class="metric-box">
    <h3>🌇 Most visited city</h3>
    <p id="top-city">Loading...</p>
  </div>
</div>

<script>
  fetch('https://api.viberadar.io/metrics/this-weeks-users').then(res => res.json()).then(data => {
    document.getElementById('this-weeks-users').textContent = data["this-weeks-users"] + ' users';
  });

  fetch('https://api.viberadar.io/metrics/avg-session-duration').then(res => res.json()).then(data => {
    document.getElementById('session-duration').textContent = Math.round(data["session-duration"]) + ' seconds';
  });

  fetch('https://api.viberadar.io/metrics/top-city').then(res => res.json()).
  then(data => {
    document.getElementById('top-city').textContent = data["top-city"];
  });
</script>

## Final Thoughts: Build, Share, Iterate

Building a cool app is only half the journey. The other half is getting it in front of people - and making sure they enjoy using it.

Here’s what worked for us:

* Add analytics, but don’t compromise user trust
* Stay legally compliant *without annoying banners*
* Continue to share and iterate
* Measure what matters and see what works

We’ll keep refining VibeRadar, and sharing what we learn. [Try it out](https://viberadar.io) and let us know what you think.

**References**

PostHog. (n.d.). [PostHog Documentation](https://posthog.com/docs).

European Union. (2002). Directive 2002/58/EC of the European Parliament and of the Council of 12 July 2002 concerning the processing of personal data and the protection of privacy in the electronic communications sector (ePrivacy Directive). [Official Journal](https://eur-lex.europa.eu/eli/dir/2002/58/oj/eng).

European Union. (2016). Regulation (EU) 2016/679 of the European Parliament and of the Council of 27 April 2016 (General Data Protection Regulation). [Official Journal](https://eur-lex.europa.eu/eli/reg/2016/679/oj/eng).

Ebert, N. (2022). The Psychology of Cookie Banners from a Data Privacy Perspective. [ZHAW Blog](https://blog.zhaw.ch/datascience/the-psychology-of-cookie-banners-from-a-data-privacy-perspective/).

PostHog. (2024). [Property Filter Transformation in PostHog](https://posthog.com/tutorials/property-filter).

VibeRadar. (2025). [Viberadar Privacy Policy](https://viberadar.io/privacy-policy).

**Keywords**
web app, analytics, PostHog, cookies, user acquisition, VibeRadar, vibe
