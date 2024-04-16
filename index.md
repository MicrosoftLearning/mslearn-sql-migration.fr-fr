---
title: Exercices Azure OpenAI
permalink: index.html
layout: home
---

# Exercices de migration de SQL Server

Les exercices suivants complètent les modules du parcours d’apprentissage [Migrer les charges de travail SQL server vers Azure SQL](https://learn.microsoft.com/training/paths/migrate-sql-workloads-azure/) sur Microsoft Learn.

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}