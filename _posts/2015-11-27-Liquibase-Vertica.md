---
layout: post
section-type: post
title: Liquibase Vertica
category: Liquibase
tags: [ 'liquibase', 'Vertica' ]
---

### Liquibase Vertica

Liquibase is a versioned schema managment tool, it let's you define and keep track of your schema changes via simple content files (change sets).

The liquibase-vertica project is an extension to liquibase that alows usage of liquibase with HP's Big-Data analytical DB.


Support the repo by starring or forking it!

<iframe src="https://ghbtns.com/github-btn.html?user=cohenjo&repo=liquibase-vertica&type=star&count=true&size=large" frameborder="0" scrolling="0" width="160px" height="30px"></iframe>

<iframe src="https://ghbtns.com/github-btn.html?user=cohenjo&repo=liquibase-vertica&type=fork&count=true&size=large" frameborder="0" scrolling="0" width="160px" height="30px"></iframe>


Sample change set definition
```XML
<changeSet author="jony" id="1">
    <ext:createTable tableName="t7">
        <column name="id" type="INT">
            <constraints primaryKey="true" primaryKeyName="C_PRIMARY"/>
        </column>
        <column name="i1" type="INT">
            <constraints nullable="false"/>
        </column>
        <column name="i2" type="INT"/>
    </ext:createTable>
</changeSet>
<changeSet author="jony" id="2">
    <ext:createProjection nodes="ALL NODES" orderby="id" projectionName="t7_super" schemaName="bla" segmentedby="hash(t7.id)" subquery="Select * from t7" ksafe="">
        <column encoding="AUTO" name="id" type="INT"/>
        <column encoding="AUTO" name="i1" type="INT"/>
        <column encoding="AUTO" name="i2" type="INT"/>
    </ext:createProjection>
</changeSet>

```
A a new post template with name YYYY-MM-DD-hello-world.md will be created under ./\_posts, with the current date.

In the created post, just replace the Title, Category and tags and you can
start writing your post in markdown right bellow the end of the post header.

Every file with the format <i>YYYY-MM-DD-post-title.md</i> will be processed as a
post, with publication date <i>YYYY-MM-DD</i>.

The content starts with:

<pre><code data-trim class="yaml">
---
layout: post
section-type: post
title: Title
category: Category
tags: [ 'tag1', 'tag2' ]
---
</code></pre>

The *layout* and *section-type* are used by the theme.

### Post navigation

You can navigate between the posts by swiping left/right in the post pages!

### Hashtags

Note: *{ Personal }* generates a static page, just like all Jekyll themes.
As a result we have to create the tag pages before building and publishing the site.

In order to generate the tag pages, simply run the *generate-tags* script from the repo's root directory:

<pre><code data-trim class="bash">
./scripts/generate-tags
</code></pre>

The script will parse all your posts, and generate the tag pages for the newly added tags.

If you are not using Github Pages, you can automate the execution of this script during build time.

### Syntax highlighting

If you want to include a code snippet in your post, simply use the following syntax:

<pre><code data-trim class="c">
int main()
{
  printf("Hello, world of syntax highlighting!");
  return 0;
}
</code></pre>

<small>If you don't need syntax highlight in your website you can disable it by setting the syntax-highlight variable to False</small>

### Emoji support

You can add emojis to your posts by simply typing their [emoji code](http://www.emoji-cheat-sheet.com/)

:wink:
