# Chrome-Extension-for-Creating-News-Drafts-in-WordPress-Custom-Post-Type-news
Objective
Develop a Chrome Extension that enables users to create draft posts in the WordPress custom post type news. The extension will fetch or generate metadata (title, tags, description, featured image) from the current webpage, set the page URL in the custom field (news_url), and allow customizable prompts for AI-generated content and images. Integration will leverage the WordPress REST API, JWT Auth plugin, OpenAI API, and DALL-E API.

Features and Requirements
1. User Interface (Chrome Extension)
Popup UI:
[ ] Create a "Create News Draft" button for the current webpage.
[ ] Display editable fields for:
[ ] Title, with button to generate
[ ] Tags input is empty with a list of suggested tags that we can choose from showing beneath (the ones which are in our news taxonomy already, will be shown in green, and new ones will be in red). We can also type manually new one into the input directly.
[ ] Description, with button to generate
[ ] Featured Image (preview or placeholder for generated image) with option to (re)generate or delete (stays empty)
[ ] Add a "Submit to WordPress" button to create the draft post.
Settings Panel:
[ ] Add fields to configure:
[ ] WordPress API URL.
[ ] JWT credentials (username/email and application password).
[ ] OpenAI API key.
[ ] DALL-E API key.
[ ] Add fields for user-configurable prompts:
[ ] Title prompt.
[ ] Tags prompt (get our existing tags and put in a variable {existing_tags}
[ ] Description prompt.
[ ] Image generation prompt (support variables like {title}, {tags}, {description}).

2. Functional Requirements
2.1 Fetch Webpage Data
[ ] Extract metadata from the current webpage:
[ ] Title: Use the <title> tag.
[ ] Tags: Extract <meta name="keywords">, and get our tags taxonomy “News Tags”
[ ] Description: Extract <meta name="description">.
[ ] Page URL: Capture the current page URL and save it to the news_url custom field.
[ ] Content of page for generating an accurate description
[ ] Use AI for (fallback) generation:
[ ] Title: Generate with OpenAI API only if the <title> tag is missing.
[ ] Tags: Generate tags with OpenAI API, based on keywords and based on our existing news tags taxonomy, suggest new ones if relevant. (This is defined in the prompt, but we need to get our tags taxonomy as input, too besides the keywords)
[ ] Description: Generate with OpenAI API 
2.3 Featured Image Handling
[ ] Attempt to fetch the Open Graph image (<meta property="og:image">).
[ ] If no image is found:
[ ] Use the DALL-E API to generate an image with a customizable prompt.
2.4 WordPress Integration
[ ] Use the WordPress REST API to:
[ ] Create a draft post in the news custom post type:
[ ] Title
[ ] Content (description)
[ ] Tags
[ ] Custom Field (news_url).
[ ] Featured Image.
[ ] Upload featured image using the WordPress Media Upload API.
2.5 Configurable AI Prompts
[ ] Allow the user to customize prompts for:
[ ] Title generation.
[ ] Tags generation.
[ ] Description generation.
[ ] Image generation.
[ ] Support placeholders for variables:
{title}, {tags}, {description}, {page_content}, {existing_tags},

3. Technical Requirements
3.1 Chrome Extension
[ ] Popup UI:
[ ] Interface for editing and submitting drafts.
[ ] Content Script:
[ ] Extract metadata and Open Graph data from the webpage.
[ ] Background Script:
[ ] Handle API requests for WordPress, OpenAI, and DALL-E.
[ ] Permissions:
[ ] activeTab for accessing the current webpage content.
[ ] storage for saving settings and tokens.
3.2 WordPress REST API
[ ] Endpoints:
[ ] POST /wp-json/wp/v2/news: Create draft posts.
[ ] POST /wp-json/wp/v2/media: Upload featured images.
[ ] Authentication:
[ ] Use JWT Auth to validate and store tokens.
3.3 OpenAI API
[ ] Use OpenAI GPT API to:
[ ] Generate titles.
[ ] Generate descriptions.
[ ] Generate tags.
3.4 DALL-E API
[ ] Use the DALL-E API to generate images if none are available.

4. Workflow
Authenticate with WordPress:
[ ] Validate credentials and store JWT token(?), or with WordPress Auth?
Extract Metadata:
[ ] Extract <title>, <meta> tags, and Open Graph data.
[ ] Fallback to OpenAI API for missing metadata or on button click to generate and replace.
Generate Image (if necessary):
[ ] Fetch Open Graph image.
[ ] Use DALL-E API with the customizable prompt for generation.
Submit Draft to WordPress:
[ ] Use the WordPress REST API to create a draft post with all data.
Feedback to User:
[ ] Provide success or error notifications.

Configuration Options in Settings Panel
WordPress Settings
[ ] WordPress site URL.
[ ] JWT Auth credentials (or Wordpress Auth):
[ ] Username/email.
[ ] Application password.
AI and Image Generation Settings
[ ] OpenAI API key.
[ ] DALL-E API key.
Custom Prompts
[ ] Title prompt.
[ ] Description prompt.
[ ] Tags prompt.
[ ] Image generation prompt.

Deliverables
[ ] Fully functional Chrome Extension with:
[ ] Popup UI for creating and submitting drafts.
[ ] Settings panel for configuration.
[ ] AI and image generation functionality with configurable prompts.
[ ] Documentation:
[ ] Installation instructions.
[ ] Setup guide for API keys and WordPress configuration.

-------------------
To develop a Chrome extension that enables users to create draft posts in the WordPress custom post type "news" and integrates with WordPress REST API, JWT Authentication, OpenAI API, and DALL-E API, we can break the project into several parts:

    Frontend Development (Chrome Extension with React.js and Manifest V3)
    Integration with WordPress REST API (JWT Authentication)
    AI and Image Generation with OpenAI API and DALL-E API
    Data Handling and Communication with WordPress Custom Post Type "News"

Part 1: Chrome Extension Frontend

First, we need to create the Chrome extension with a React.js frontend and Manifest V3 to interact with the WordPress site and APIs.
Step 1: Set up the React-based Chrome Extension

    Initialize the Project:

npm create vite@latest wordpress-extension --template react
cd wordpress-extension
npm install

    Manifest File (manifest.json): This manifest file configures the Chrome extension, defining permissions, content scripts, and background services.

{
  "manifest_version": 3,
  "name": "WordPress Draft Post Creator",
  "description": "Create draft posts in WordPress with AI-generated content",
  "version": "1.0",
  "permissions": ["storage", "activeTab", "identity", "https://your-wordpress-site.com/*"],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"]
    }
  ],
  "action": {
    "default_popup": "popup.html"
  },
  "oauth2": {
    "client_id": "YOUR_OAUTH2_CLIENT_ID",
    "scopes": ["email", "profile"]
  }
}

    React Components:

    Popup Component (Popup.jsx): The extension popup that allows users to interact with the page.
    Content Script (content.js): Fetches metadata (title, tags, description, and featured image).
    Background Script (background.js): Handles communication with the WordPress API.

Part 2: Integrating with WordPress REST API

The Chrome extension will use WordPress's REST API for creating the custom post type "news" in WordPress. We'll authenticate using JWT and send the metadata to WordPress.

    JWT Authentication for WordPress REST API: Install the JWT Authentication plugin for WordPress to handle JWT authentication.

    Authenticate User in the Extension:

To authenticate the user, we'll use the OAuth2 mechanism for getting the JWT token from the WordPress backend.

chrome.identity.getAuthToken({interactive: true}, function(token) {
  // Send token to WordPress API for authentication
  fetch('https://your-wordpress-site.com/wp-json/wp/v2/users/me', {
    method: 'GET',
    headers: {
      'Authorization': 'Bearer ' + token
    }
  })
  .then(response => response.json())
  .then(user => {
    console.log("Authenticated User: ", user);
  })
  .catch(error => console.error('Authentication Error: ', error));
});

    Create the Custom Post Type "News" in WordPress:

After authentication, send a POST request to create a draft post in the custom post type "news".

function createWordPressPost(postData, token) {
  fetch('https://your-wordpress-site.com/wp-json/wp/v2/news', {
    method: 'POST',
    headers: {
      'Authorization': 'Bearer ' + token,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(postData)
  })
  .then(response => response.json())
  .then(data => {
    console.log("Post Created:", data);
  })
  .catch(error => console.error('Post Creation Error: ', error));
}

The postData object would look like this:

const postData = {
  title: "Generated Title",
  content: "Generated Content from AI",
  status: "draft",
  fields: {
    news_url: window.location.href // Set current page URL in the custom field
  },
  tags: ["AI", "Technology", "News"],
  featured_media: imageID // The media ID of the uploaded image (generated by DALL-E)
};

Part 3: AI Content and Image Generation with OpenAI and DALL-E

To generate AI-driven content and images, we'll use OpenAI's GPT API for content and DALL-E API for image generation.

    Generate AI Content with OpenAI:

const generateAIContent = async (pageContent) => {
  const response = await fetch('https://api.openai.com/v1/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer YOUR_OPENAI_API_KEY`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      model: 'text-davinci-003',
      prompt: `Generate a news article summary about this content: ${pageContent}`,
      max_tokens: 200
    })
  });
  const data = await response.json();
  return data.choices[0].text;
}

    Generate Image with DALL-E:

const generateImage = async (prompt) => {
  const response = await fetch('https://api.openai.com/v1/images/generations', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer YOUR_DALLE_API_KEY`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      prompt: prompt,
      n: 1,
      size: "1024x1024"
    })
  });
  const data = await response.json();
  return data.data[0].url; // Image URL
}

Part 4: Handling Metadata from Current Page

The content script will extract metadata such as title, description, tags, and image from the current page.

// content.js
function getPageMetadata() {
  const title = document.title;
  const description = document.querySelector('meta[name="description"]')?.content || '';
  const image = document.querySelector('meta[property="og:image"]')?.content || '';
  const tags = document.querySelector('meta[name="keywords"]')?.content.split(',') || [];

  return { title, description, image, tags };
}

// Send metadata to background script for processing
chrome.runtime.sendMessage({
  action: "processMetadata",
  metadata: getPageMetadata()
});

Background Script

The background script will handle sending the metadata to WordPress, generating content via OpenAI, and generating images with DALL-E.

// background.js
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === "processMetadata") {
    const metadata = message.metadata;

    // Generate AI Content
    generateAIContent(metadata.description).then(content => {
      // Generate Image with DALL-E
      generateImage(metadata.description).then(imageUrl => {
        // Create post data
        const postData = {
          title: metadata.title,
          content: content,
          status: "draft",
          fields: { news_url: window.location.href },
          tags: metadata.tags,
          featured_media: imageUrl // URL to the generated image
        };

        // Fetch the token and send the post data to WordPress
        chrome.identity.getAuthToken({interactive: true}, function(token) {
          createWordPressPost(postData, token);
        });
      });
    });
  }
});

Step 5: Testing and Deployment

    Test the Extension Locally: Load the unpacked extension via chrome://extensions, then test its functionality in the browser.

    Deploy the Extension: Once everything works, you can package the extension and upload it to the Chrome Web Store.

Summary

This Chrome extension allows users to generate draft posts in the WordPress custom post type "news" by automatically fetching metadata from the current page, generating AI content using OpenAI, and generating a featured image using DALL-E. The extension integrates with WordPress using the REST API and JWT Authentication, providing a seamless experience for users to create and manage content.
To implement the Chrome Extension as described in the provided objective, we need to break it down into clear, manageable steps. Here’s a comprehensive overview of the required features and technical implementation:
1. User Interface (Chrome Extension)
Popup UI

The popup will provide the following UI elements:

    Create News Draft Button: A button to create a new draft post using the data from the current webpage.
    Editable Fields:
        Title: Text input with an option to generate a title using OpenAI.
        Tags: Input field with suggestions from existing WordPress tags (shown in green for existing, red for new). Ability to add custom tags.
        Description: A text area with a button to generate a description using OpenAI.
        Featured Image: Display a placeholder or preview for the generated image with an option to regenerate or delete it.
    Submit to WordPress Button: A button to submit the draft to the WordPress site.

Settings Panel

The settings panel will allow users to configure:

    WordPress API URL: Set the base URL for the WordPress REST API.
    JWT Credentials: Username/email and application password for JWT authentication.
    API Keys:
        OpenAI API Key
        DALL-E API Key
    Customizable Prompts:
        Title Prompt
        Tags Prompt (using existing tags)
        Description Prompt
        Image Generation Prompt (using variables like {title}, {tags}, {description})

2. Functional Requirements
2.1 Fetch Webpage Data

    Title: Extract from the <title> tag.
    Tags: Extract tags from <meta name="keywords"> and the WordPress "News Tags" taxonomy.
    Description: Extract from <meta name="description">.
    Page URL: Capture the current URL and save it in the news_url custom field.

2.2 AI Content Generation

    Fallback for Missing Metadata:
        Title: Use OpenAI if the title is missing.
        Tags: Generate tags using OpenAI based on the webpage content and existing tags.
        Description: Generate description using OpenAI API based on the page content.

2.3 Featured Image Handling

    Attempt to fetch the Open Graph image (<meta property="og:image">).
    If no image is found, use the DALL-E API to generate an image with a customizable prompt.

2.4 WordPress Integration

    Create Draft Post: Use the WordPress REST API to create a draft post with the fetched metadata (title, tags, description, featured image).
    Upload Featured Image: Use the WordPress Media Upload API to upload the generated or fetched featured image.

2.5 Configurable AI Prompts

    Allow users to customize the prompts for:
        Title generation
        Tags generation
        Description generation
        Image generation

3. Technical Implementation
3.1 Chrome Extension

    Manifest File (manifest.json)

{
  "manifest_version": 3,
  "name": "WordPress Draft Post Creator",
  "description": "Create draft posts in WordPress with AI-generated content",
  "version": "1.0",
  "permissions": ["storage", "activeTab", "identity", "https://your-wordpress-site.com/*"],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"]
    }
  ],
  "action": {
    "default_popup": "popup.html"
  },
  "oauth2": {
    "client_id": "YOUR_OAUTH2_CLIENT_ID",
    "scopes": ["email", "profile"]
  }
}

    Popup Component (popup.jsx)

import React, { useState, useEffect } from 'react';

function Popup() {
  const [title, setTitle] = useState('');
  const [tags, setTags] = useState('');
  const [description, setDescription] = useState('');
  const [featuredImage, setFeaturedImage] = useState('');

  useEffect(() => {
    // Fetch saved settings like WordPress API URL, API keys
  }, []);

  const handleGenerateContent = async () => {
    // Use OpenAI API to generate title, description, and tags based on webpage metadata
  };

  const handleSubmitPost = async () => {
    // Call the background script to create the post in WordPress
  };

  return (
    <div>
      <button onClick={handleGenerateContent}>Generate Draft</button>
      <input type="text" value={title} onChange={(e) => setTitle(e.target.value)} />
      <input type="text" value={tags} onChange={(e) => setTags(e.target.value)} />
      <textarea value={description} onChange={(e) => setDescription(e.target.value)} />
      <img src={featuredImage} alt="Featured" />
      <button onClick={handleSubmitPost}>Submit to WordPress</button>
    </div>
  );
}

export default Popup;

    Content Script (content.js)

function getPageMetadata() {
  const title = document.title;
  const description = document.querySelector('meta[name="description"]')?.content || '';
  const image = document.querySelector('meta[property="og:image"]')?.content || '';
  const tags = document.querySelector('meta[name="keywords"]')?.content.split(',') || [];

  return { title, description, image, tags };
}

chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.action === 'extractMetadata') {
    sendResponse(getPageMetadata());
  }
});

    Background Script (background.js)

chrome.runtime.onMessage.addListener(async (message, sender, sendResponse) => {
  if (message.action === 'createPost') {
    const { title, content, tags, image } = message.postData;
    const token = await getAuthToken();

    const postData = {
      title,
      content,
      status: 'draft',
      fields: { news_url: window.location.href },
      tags,
    };

    const postResponse = await createWordPressPost(postData, token);
    if (image) {
      const imageResponse = await uploadImage(image, token);
      postData.featured_media = imageResponse.id;
    }

    sendResponse({ success: true, post: postResponse });
  }
});

async function getAuthToken() {
  // Fetch the JWT token from localStorage or authenticate with WordPress
}

async function createWordPressPost(data, token) {
  return fetch('https://your-wordpress-site.com/wp-json/wp/v2/news', {
    method: 'POST',
    headers: { 'Authorization': 'Bearer ' + token, 'Content-Type': 'application/json' },
    body: JSON.stringify(data),
  }).then(response => response.json());
}

async function uploadImage(imageUrl, token) {
  const imageData = await fetch(imageUrl).then(res => res.blob());
  const formData = new FormData();
  formData.append('file', imageData, 'featured_image.jpg');

  return fetch('https://your-wordpress-site.com/wp-json/wp/v2/media', {
    method: 'POST',
    headers: { 'Authorization': 'Bearer ' + token },
    body: formData,
  }).then(response => response.json());
}

3.2 OpenAI and DALL-E API Integration

    OpenAI API for Content Generation:

const generateContentWithOpenAI = async (prompt) => {
  const response = await fetch('https://api.openai.com/v1/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer YOUR_OPENAI_API_KEY`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      model: 'text-davinci-003',
      prompt,
      max_tokens: 200,
    }),
  });
  const data = await response.json();
  return data.choices[0].text;
};

    DALL-E API for Image Generation:

const generateImageWithDalle = async (prompt) => {
  const response = await fetch('https://api.openai.com/v1/images/generations', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer YOUR_DALLE_API_KEY`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      prompt,
      n: 1,
      size: '1024x1024',
    }),
  });
  const data = await response.json();
  return data.data[0].url; // Return the image URL
};

4. Workflow

    Authentication: Use JWT authentication to secure communication with WordPress.
    Extract Webpage Data: Use content script to extract metadata and Open Graph data.
    Generate AI Content: Use OpenAI to generate content and tags.
    Generate Image: Use DALL-E if no image is found on the page.
    Create Draft Post: Submit all the data to WordPress via the REST API.

5. Deliverables

    Fully functional Chrome Extension with:
        Popup UI for creating and submitting drafts.
        Settings panel for configuration.
        AI and image generation functionality with customizable prompts.
    Documentation:
        Installation instructions.
        Setup guide for API keys and WordPress configuration.

This implementation ensures that users can efficiently create and submit draft posts to WordPress with metadata extracted from the current page, AI-generated content, and images, all through a seamless Chrome Extension interface.
