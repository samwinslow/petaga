# Petaga
The code that makes Petaga tick

## Build Instructions
You will need to install Mustache first in order to render the templates, like this:
```
gem install mustache
```
`git clone` this repository and `cd` into it; then run the server and build script. The ampersand is optional, leave it out if you want verbose output of all requests. If you leave it in, python will run in the background and you will need to `kill` the process manually.
```
python -m SimpleHTTPServer&
./build.sh
```
### For backend testing
For testing of the PHP (for now) backend, which is not available through GitHub for security reasons, create (or download the working copy of) a folder separate from the front-end repository and run the following command, provided you have PHP ≥5.4.0:
```
php -S localhost:<PORT>
```
Be sure to replace `<PORT>` with a port other than standard reserved ports or 8000, which is used by the Python server for the frontend. This section will be more complete in a few days.

## Spec
This section is intended to give very general background information and to function as a guide to start development on the re-writing of this project. In it are the general concepts that will help us get started.

### Switching to Git
- Use of Git and GitHub is important for expansion and collaboration from the community and potential employees
- Foreseeable initial drawbacks to using GitHub include:
  - Exposure of database passwords, as the current system uses PHP with MySQL passwords listed as plaintext
  - Code propriety and risk of theft/out-competition
  - Security holes are open and exposed
- Learning to use Git: http://www.railstutorial.org/book/beginning#sec-version_control

### Solution
Uses GitHub and maintains security:

| Frontend | Backend |
| -------- | ------- |
| Can be simple Javascript | Traditional MySQL and PHP |
| None/very few embedded PHP & MySQL calls (better for security) | Receives API calls |
| **Open** for forks and contributions through Github | **Secure** code that is developed privately and not shared |
  
### New structure
Note that the terms frontend and backend are being used very loosely here to make a clear distinction between which side makes an XHR request and which side processes it. The flow of information will work like this (it is very simple)

1. User submits a request to the frontend by visiting a page.
2. Static resources (Stylesheets, scripts, images) are pulled in as the page loads either from the frontend or from a separate CDN.
3. Through JavaScript, the page submits an XMLHttpRequest to the backend which will return relevant requested data.
4. PHP code (or code in any other language) processes the API call, talks to the database, and gets the data.
5. PHP code returns JSON data to the frontend to be digested by a callback function and shown to the user.

### Advantages of this structure
- Frontend and backend are independent
  - Can even be hosted on separate servers, perfect for scalability and reduced load
  - Frontend and backend are language-independent too, since everything is done through API calls. This makes it easier to change the markup of the frontend or the structure of the backend without re-doing both.
- No sensitive information is shared through GitHub
  - To access the Administration panel of the webapp, credentials are required to be sent to the backend (non-open-source) side where verification takes place
- Forks of the frontend code are essentially required to use our API unless they construct their own from scratch.
  - All distinct information stored in backend

### Language and structure specifics
- Frontend (View and Controller)
  - Primary view, built from basic partials, generated with compiled Mustache templating engine so that logic is mostly separate and things can be changed easily
  - Starting with Boilerplate and Bootstrap for default CSS and JavaScript
  - Static resources such as CSS and non-changing images are pulled from the frontend
  - A single JavaScript file controls logic and defines functions; client-side mustache templates may be used for this
  - Rewrite engine TBD
- Backend (Data Model)
  - Python and PostgreSQL (or PHP and MySQL, it doesn't matter much) which will return JSON-formatted data to the view.
  - Stores images and relevant data which can be retrieved by Python

## Code Documentation
Documentation outlining the intricacies of our own custom implementation of Mustache.

### Where, how, and why Mustache is used
- **Where**
  - Both in pre-rendered templates and on the client side
- **How**
  - Templates are pre-rendered for the sole purpose of including partials. No complex or dynamic views should be included here.
  - Dynamic data retrieved from the API/backend is included client-side. Read the next section for details.
- **Why**
  - Mustache is used to help us separate logic from views, thereby helping create cleaner, more modular, and more maintenance-friendly code.

### Syntax for pre-rendering versus client-side
You are probably used to the normal Mustache syntax, like this:
```
{{#data}}
  <li>{{title}}</li>
{{/data}}
```
Presumably the above code would be wrapped in a list and would iterate through an array or object named `data` returning each `title`. But it is not possible with regular Mustache to do the sort of pre-rendering and client-side rendering we need, so the above code would be applicable _only_ during pre-rendering by build.sh. And at any rate, iterating through this kind of data may very well involve data that comes from the backend, and pre-rendering wouldn't make sense for a dynamic view. The only kind of information to render with pre-rendered Mustache would be that which is static and will probably not change. Therefore the most common directives in the mustache templates will probably be partials, similar to these:
```
{{> header }}
  <h1>Our great page title</h1>
  <p>A little more to tell you what it's all about. The rest of the page's body content goes here. The markup has clearly been over-simplified!</p>
{{> footer }}
```
Revisiting the first example, again it is much better to pull data from our server and render on the client-side like this:
```
<div class="view" data-template="test1">
  <ul>
    [[#data]]
    <li>[[title]]</li>
    [[/data]]
  </ul>
</div>
```
Note especially that the content to be rendered is placed into the document surrounded by a `div.view` and that where curly braces (mustaches?) would have been used, brackets are used instead. Documentation is coming soon on the `data-template` attribute.

