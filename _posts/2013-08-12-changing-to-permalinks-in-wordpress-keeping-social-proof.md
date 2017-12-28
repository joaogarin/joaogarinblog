---
layout: post
title: "Changing to permalinks in wordpress keeping social proof"
description: "One of the things to worry when chaning URL's in any website is keeping social proof in past shared links."
thumb_image: "posts/social-media-mess.jpg"
tags: [web]
---

{% include image.html path="posts/social-media-mess.jpg" path-detail="posts/social-media-mess.jpg" alt="WP social links" %}

One of the things to worry when chaning URL's in any website is keeping social proof in past shared links, keeping sure that these are still valid and still counted when users visit a page.

Plugins like digg digg, or sharethis help us with this but the way they worked is based on the exact URL that was shared. So for example if someone has 100 shares on facebook on a blog post for example if they change URL these buttons will be down to zero again.

The trick is to keep post the "old URL" even if the URL has changed.

I had recently this situation in wordpress and I decided to post the solution here for the plugin I was using (digg digg) as I tried to find an easy solution online but found none.

So I changed the URL's from the default wordpress URL (?p=post_id) to a very simple (/post-name). The trick was hacking the file digg-digg.php in function dd_hook_wp_content . Chaging to the following lines :

{% highlight php %}
global $wp_query;
$post = $wp_query-&gt;post; //get post content
$id = $post-&gt;ID; //get post id
$postlink = get_permalink($id); //get post link
$commentcount = $post-&gt;comment_count; //get post comment count
$title = trim($post-&gt;post_title); // get post title
<strong>//HACK BY JOAO</strong>
$sharing_url = get_permalink();
<strong>//First post id after changing to permalinks</strong>
$url_change_id = "1020";
$post_id = $id;
if ($post_id &lt; $url_change_id) {
  $url_id_postfix = "/?p=" . $post_id;
  $sharing_url = "http://example.com".$url_id_postfix;
}
$link = explode(DD_DASH,$postlink); //split the link with '#'
$url = $link[0];
$url = $sharing_url;
{% endhighlight %}

So this hopefully fixes it for you guys, any question feel free to ask!.
