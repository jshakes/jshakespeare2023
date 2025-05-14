---
date: 2025-05-14 05:00:52+00:00
title: Fix Missing ‘Skip to Content’ Link in WordPress Block Themes
description:
featured_image:
featured_image_caption:
featured_image_alt:
slug: fix-missing-skip-to-content-link-wordpress-block-themes
---

WordPress sites are steadily becoming more accessible out of the box with the increasing adoption of [Full Site Editing \(FSE\)](https://fullsiteediting.com/lessons/what-is-full-site-editing/). This is great news for the web, as it reduces the responsibility of making sites accessible on individual developers and site owners, shifting more of that burden onto the platform itself.

One a11y feature that is available by default in WordPress 5.8 and later is the automatic addition of ’Skip to content’ links, known as skip-links, to block theme templates. Skip-links help keyboard and screen reader users quickly navigate to the main content area of a page and avoid repetitive tabbing through header and navigation areas above it.

You can test whether skip-links are working on your WordPress FSE site by loading a page and pressing `Tab`. You should see a link appear in the top-left corner of the page saying **“Skip to content.”** Pressing `Enter` while this link is in focus should shift focus past the navigation and into the main content area.

If you don’t see the skip link, it’s likely that your template is missing a `<main>` element.

---

## How WordPress determines whether and where to add a skip link

If you view the source of your page, you will see see the following snippet, which is added by `[wp_enqueue_block_template_skip_link](https://github.com/WordPress/WordPress/blob/6.4/wp-includes/theme-templates.php#L109C10-L109C45)`:

```html
<script id="wp-block-template-skip-link-js-after">
( function() {
	var skipLinkTarget = document.querySelector( 'main' ),
		sibling,
		skipLinkTargetID,
		skipLink;

	// Early exit if a skip-link target can't be located.
	if ( ! skipLinkTarget ) {
		return;
	}

	/*
	 * Get the site wrapper.
	 * The skip-link will be injected at the beginning of it.
	 */
	sibling = document.querySelector( '.wp-site-blocks' );

	// Early exit if the root element was not found.
	if ( ! sibling ) {
		return;
	}

	// Get the skip-link target's ID, and generate one if it doesn't exist.
	skipLinkTargetID = skipLinkTarget.id;
	if ( ! skipLinkTargetID ) {
		skipLinkTargetID = 'wp--skip-link--target';
		skipLinkTarget.id = skipLinkTargetID;
	}

	// Create the skip link.
	skipLink = document.createElement( 'a' );
	skipLink.classList.add( 'skip-link', 'screen-reader-text' );
	skipLink.id = 'wp-skip-link';
	skipLink.href = '#' + skipLinkTargetID;
	skipLink.innerText = 'Skip to content';

	// Inject the skip link.
	sibling.parentElement.insertBefore( skipLink, sibling );
}() );
</script>
```

Notice that if no main element is found (`skipLinkTarget`), the script exits early, meaning no skip link will be added.

# How to add a<main> element in the Site Editor
1. Open the WordPress Editor and navigate to the relevant template (e.g. “Page” or “Single”).
2. Locate or create a **Group** block that wraps all of your primary content. Ideally, this Group should contain your page’s <h1> heading as its first child.
3. If no suitable Group exists, select all the relevant blocks and press Cmd + G (or Ctrl + G on Windows) to group them.
4. With the Group block selected, open the **Block settings** sidebar.
5. Under the **Advanced** section, find the **HTML element** dropdown and select <main>.
6. Save the template.
7. Refresh the page. You should now see the “Skip to content” link appear as the first item to receive focus when pressing Tab.
8. Repeat this process for any other templates that are missing a skip-link.
