---
title: 'Super De-Duper: Identifying duplicates in messy data'
subtitle: 
summary: Super De-Duper is the data product I built in 4 weeks during the insight data science fellowship
authors:

tags:
- Academic

categories:
date: 
lastmod: 
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Placement options: 1 = Full column width, 2 = Out-set, 3 = Screen-width
# Focal point options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
image:
  placement: 2
  caption: 'Image credit: [**Unsplash**](https://unsplash.com/photos/CpkOjOcXdUY)'
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---


As we data scientist know, the quality of the data can make or break a data product. Poor quality data has led to a loss of ~600B US dollars just in the field of marketing alone. Another report cuts duplicate data as the mother of all data quality problems. Duplicate data issues are faced by many companies whose customers are B2B companies. They get their data from different CRMs and accounting systems which has data in different formats. Duplicate records when not removed can negatively affect data products such as customer analytics - customer segmentation analysis, in the worst case can led the same customer to be in several tiers. The goal of Super De-Duper is to identify duplicates records and provide a “Dupe” score, a custom metric that quantifies the duplicity of the records. 

I consulted with a startup company in Boston, Tally Street, who provide virtual accountant services to B2B companies and worked on their customer profile data to remove duplicates. The main features of the data that I used for this project are: name of the company, their address and primary phone number. There are several duplicates in the dataset, one example is shown below:

| Full name      | Company name | Address line1 | Address line 2 | City | State | Zipcode | Phone number |
| ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- |
| Curtis Corrado Insurance Agency (deleted) | Curtis Corrado Insurance Agency | 5975 S Quebec St. | Suite 209 | Centennial | CO | 80111 | (303) 220-7688 |
| Curtis Corrado Insurance Agency | Farmers Insurance-Curt Corrado Agency | 5975 S Quebec St | #209 | Centennial | CO | 80111 | 3032207688 |

Just by looking at these records, we can say that the second record is a duplicate of the first, but this is not an easy task for computer.

## Data Challenges:

With an end goal of identifying duplicates, there are two main challenges I faced in this dataset. They are (a) very messy data and (b) >50% missing data. The data, particularly in the address fields such as city, state and country is very messy. For example, a snapshot of the unique value observed in the billing address feature “country” is shown below: 




For a feature named country, we would expect the feature to contain name of the countries, for example - USA, Mexico. But, this feature, in addition to the country name, also contains other metadata such as city name, company name, phone number, email address, so on. It should be noted that even though we have separate fields for them, the data is pretty much jumbled around all the fields. 


The second challenge to attain our end goal of finding duplicates is - missing data. More than 50% of the data is missing in this data set. Even after data cleaning (will be explained later), only 42% of the data has non-missing values in all features. 

## Approach:

Considering the diverse nature of data in the three main features (name, address and phone number) together with data challenges mentioned above, I decided to create three different pipeline for each of these features to overcome these challenges.


1. Data pipeline for Company name: 
	
  Three columns were found for company names in the dataset - display name, fully qualified name and company name. 

Python code here to show display name is redundant

	From analysis it was clear that the display name is a redudant feature, and thus, it was removed. The other two name features are retained as they provide varied information. 

	After feature selection, the data is converted into vector using TF-IDF vectorization with character embedding. There are duplicates in the dataset because of spelling error, that I chose to perform character embedding. Also, lot of companies has ‘ltd’ and ‘Co’ at the end and TF-IDF gives less importance to that. 
	
	Finally to find duplicate names in the dataset, cosine similarity is chosen. Note that the char embedding resulted in 300K+ features, where Euclidean distance will fail. 



**Create a free website with Academic using Markdown, Jupyter, or RStudio. Choose a beautiful color theme and build anything with the Page Builder - over 40 _widgets_, _themes_, and _language packs_ included!**

[Check out the latest **demo**](https://academic-demo.netlify.com/) of what you'll get in less than 10 minutes, or [view the **showcase**](https://sourcethemes.com/academic/#expo) of personal, project, and business sites.

- 👉 [**Get Started**](#install)
- 📚 [View the **documentation**](https://sourcethemes.com/academic/docs/)
- 💬 [**Ask a question** on the forum](https://discourse.gohugo.io)
- 👥 [Chat with the **community**](https://spectrum.chat/academic)
- 🐦 Twitter: [@source_themes](https://twitter.com/source_themes) [@GeorgeCushen](https://twitter.com/GeorgeCushen) [#MadeWithAcademic](https://twitter.com/search?q=%23MadeWithAcademic&src=typd)
- 💡 [Request a **feature** or report a **bug**](https://github.com/gcushen/hugo-academic/issues)
- ⬆️ **Updating?** View the [Update Guide](https://sourcethemes.com/academic/docs/update/) and [Release Notes](https://sourcethemes.com/academic/updates/)
- :heart: **Support development** of Academic:
  - ☕️ [**Donate a coffee**](https://paypal.me/cushen)
  - 💵 [Become a backer on **Patreon**](https://www.patreon.com/cushen)
  - 🖼️ [Decorate your laptop or journal with an Academic **sticker**](https://www.redbubble.com/people/neutreno/works/34387919-academic)
  - 👕 [Wear the **T-shirt**](https://academic.threadless.com/)
  - :woman_technologist: [**Contribute**](https://sourcethemes.com/academic/docs/contribute/)

{{< figure src="https://raw.githubusercontent.com/gcushen/hugo-academic/master/academic.png" title="Academic is mobile first with a responsive design to ensure that your site looks stunning on every device." >}}

**Key features:**

- **Page builder** - Create *anything* with [**widgets**](https://sourcethemes.com/academic/docs/page-builder/) and [**elements**](https://sourcethemes.com/academic/docs/writing-markdown-latex/)
- **Edit any type of content** - Blog posts, publications, talks, slides, projects, and more!
- **Create content** in [**Markdown**](https://sourcethemes.com/academic/docs/writing-markdown-latex/), [**Jupyter**](https://sourcethemes.com/academic/docs/jupyter/), or [**RStudio**](https://sourcethemes.com/academic/docs/install/#install-with-rstudio)
- **Plugin System** - Fully customizable [**color** and **font themes**](https://sourcethemes.com/academic/themes/)
- **Display Code and Math** - Code highlighting and [LaTeX math](https://en.wikibooks.org/wiki/LaTeX/Mathematics) supported
- **Integrations** - [Google Analytics](https://analytics.google.com), [Disqus commenting](https://disqus.com), Maps, Contact Forms, and more!
- **Beautiful Site** - Simple and refreshing one page design
- **Industry-Leading SEO** - Help get your website found on search engines and social media
- **Media Galleries** - Display your images and videos with captions in a customizable gallery
- **Mobile Friendly** - Look amazing on every screen with a mobile friendly version of your site
- **Multi-language** - 15+ language packs including English, 中文, and Português
- **Multi-user** - Each author gets their own profile page
- **Privacy Pack** - Assists with GDPR
- **Stand Out** - Bring your site to life with animation, parallax backgrounds, and scroll effects
- **One-Click Deployment** - No servers. No databases. Only files.

## Themes

Academic comes with **automatic day (light) and night (dark) mode** built-in. Alternatively, visitors can  choose their preferred mode - click the sun/moon icon in the top right of the [Demo](https://academic-demo.netlify.com/) to see it in action! Day/night mode can also be disabled by the site admin in `params.toml`.

[Choose a stunning **theme** and **font**](https://sourcethemes.com/academic/themes/) for your site. Themes are fully [customizable](https://sourcethemes.com/academic/docs/customization/#custom-theme).

## Ecosystem

* **[Academic Admin](https://github.com/sourcethemes/academic-admin):** An admin tool to import publications from BibTeX or import assets for an offline site
* **[Academic Scripts](https://github.com/sourcethemes/academic-scripts):** Scripts to help migrate content to new versions of Academic

## Install

You can choose from one of the following four methods to install:

* [**one-click install using your web browser (recommended)**](https://sourcethemes.com/academic/docs/install/#install-with-web-browser)
* [install on your computer using **Git** with the Command Prompt/Terminal app](https://sourcethemes.com/academic/docs/install/#install-with-git)
* [install on your computer by downloading the **ZIP files**](https://sourcethemes.com/academic/docs/install/#install-with-zip)
* [install on your computer with **RStudio**](https://sourcethemes.com/academic/docs/install/#install-with-rstudio)

Then [personalize and deploy your new site](https://sourcethemes.com/academic/docs/get-started/).

## Updating

[View the Update Guide](https://sourcethemes.com/academic/docs/update/).

Feel free to *star* the project on [Github](https://github.com/gcushen/hugo-academic/) to help keep track of [updates](https://sourcethemes.com/academic/updates).

## License

Copyright 2016-present [George Cushen](https://georgecushen.com).

Released under the [MIT](https://github.com/gcushen/hugo-academic/blob/master/LICENSE.md) license.
