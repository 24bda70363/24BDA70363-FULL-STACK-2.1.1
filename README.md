# 24BDA70363-FULL-STACK-2.1.1

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>RESTful API CRUD Operations</title>

<style>
body {
    font-family: Arial, sans-serif;
    background: #eef2f7;
    padding: 20px;
}

.container {
    max-width: 900px;
    margin: auto;
    background: white;
    padding: 25px;
    border-radius: 10px;
}

h2 {
    color: #1d4ed8;
}

input, textarea {
    width: 95%;
    padding: 12px;
    margin: 8px 0;
    border: 1px solid #ccc;
    border-radius: 5px;
}

textarea {
    height: 90px;
}

button {
    background: #2563eb;
    color: white;
    padding: 11px 16px;
    margin: 5px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

button:hover {
    background: #1e40af;
}

.delete {
    background: #dc2626;
}

.edit {
    background: #059669;
}

.post {
    background: #f8fafc;
    padding: 15px;
    margin: 12px 0;
    border: 1px solid #ddd;
    border-radius: 8px;
    overflow-wrap: anywhere;
}

.output {
    background: #172033;
    color: #8fffa6;
    padding: 15px;
    min-height: 140px;
    border-radius: 6px;
    white-space: pre-wrap;
    overflow-wrap: anywhere;
}

.error {
    color: #dc2626;
    font-weight: bold;
}

.success {
    color: #059669;
    font-weight: bold;
}
</style>
</head>

<body>

<div class="container">

<h2>RESTful API CRUD Operations</h2>
<p>Post Management System with Validation</p>

<h3>Create / Update Post</h3>

<input type="hidden" id="postId">

<input type="text" id="title"
placeholder="Enter Post Title">

<textarea id="content"
placeholder="Enter Post Content"></textarea>

<input type="text" id="author"
placeholder="Enter Author Name">

<br>

<button onclick="savePost()">Save Post</button>

<button onclick="clearForm()">Clear Form</button>

<p id="message"></p>

<hr>

<h3>Search Posts</h3>

<input type="text" id="search"
placeholder="Search by title or author"
oninput="displayPosts()">

<h3>All Posts</h3>

<button onclick="displayPosts()">GET All Posts</button>

<div id="postList"></div>

<hr>

<h3>API Response (JSON)</h3>
<div class="output" id="output">
API Response will appear here.
</div>

</div>

<script>

let posts = JSON.parse(
    localStorage.getItem("crudPosts") || "[]"
);

function saveData() {
    localStorage.setItem(
        "crudPosts",
        JSON.stringify(posts)
    );
}

function showResponse(status, message, data) {

    let response = {
        success: status >= 200 && status < 300,
        status: status,
        message: message,
        data: data,
        timestamp: new Date().toISOString()
    };

    document.getElementById("output").textContent =
        JSON.stringify(response, null, 4);
}

function showMessage(text, isError = false) {

    let msg = document.getElementById("message");

    msg.textContent = text;

    msg.className = isError ? "error" : "success";
}

function savePost() {

    let id = document.getElementById("postId").value;

    let title = document.getElementById("title")
        .value.trim();

    let content = document.getElementById("content")
        .value.trim();

    let author = document.getElementById("author")
        .value.trim();

    if (title.length < 3 || title.length > 150) {

        showMessage(
            "Title must be between 3 and 150 characters.",
            true
        );

        showResponse(
            400,
            "Validation Failed",
            { title: "Invalid title length" }
        );

        return;
    }

    if (content.length < 10) {

        showMessage(
            "Content must contain at least 10 characters.",
            true
        );

        showResponse(
            400,
            "Validation Failed",
            { content: "Content too short" }
        );

        return;
    }

    if (!author) {

        showMessage("Author is required.", true);

        showResponse(
            400,
            "Validation Failed",
            { author: "Author cannot be blank" }
        );

        return;
    }

    if (id) {

        let post = posts.find(p => p.id == id);

        if (!post) {
            showResponse(404, "Post Not Found", null);
            return;
        }

        post.title = title;
        post.content = content;
        post.author = author;

        showResponse(
            200,
            "Post updated successfully",
            post
        );

        showMessage("Post updated successfully!");

    } else {

        let post = {
            id: Date.now(),
            title: title,
            content: content,
            author: author,
            createdAt: new Date().toISOString()
        };

        posts.push(post);

        showResponse(
            201,
            "Post created successfully",
            post
        );

        showMessage("Post created successfully!");
    }

    saveData();
    clearForm(false);
    renderPosts();
}

function displayPosts() {

    let search = document.getElementById("search")
        .value.toLowerCase();

    let filtered = posts.filter(post =>
        post.title.toLowerCase().includes(search) ||
        post.author.toLowerCase().includes(search)
    );

    showResponse(
        200,
        "Posts fetched successfully",
        filtered
    );

    renderPosts();
}

function renderPosts() {

    let list = document.getElementById("postList");

    let search = document.getElementById("search")
        .value.toLowerCase();

    list.innerHTML = "";

    let filtered = posts.filter(post =>
        post.title.toLowerCase().includes(search) ||
        post.author.toLowerCase().includes(search)
    );

    if (filtered.length === 0) {
        list.textContent = "No posts found.";
        return;
    }

    filtered.forEach(post => {

        let div = document.createElement("div");
        div.className = "post";

        let heading = document.createElement("h3");
        heading.textContent = post.title;

        let details = document.createElement("p");
        details.textContent =
            "ID: " + post.id +
            " | Author: " + post.author;

        let body = document.createElement("p");
        body.textContent = post.content;

        let viewButton = document.createElement("button");
        viewButton.textContent = "GET";
        viewButton.onclick = () => getPost(post.id);

        let editButton = document.createElement("button");
        editButton.textContent = "Edit";
        editButton.className = "edit";
        editButton.onclick = () => editPost(post.id);

        let deleteButton = document.createElement("button");
        deleteButton.textContent = "Delete";
        deleteButton.className = "delete";
        deleteButton.onclick = () => deletePost(post.id);

        div.append(
            heading,
            details,
            body,
            viewButton,
            editButton,
            deleteButton
        );

        list.appendChild(div);
    });
}

function getPost(id) {

    let post = posts.find(p => p.id === id);

    if (post) {

        showResponse(
            200,
            "Post fetched successfully",
            post
        );

    } else {

        showResponse(404, "Post Not Found", null);
    }
}

function editPost(id) {

    let post = posts.find(p => p.id === id);

    if (!post) return;

    document.getElementById("postId").value = post.id;
    document.getElementById("title").value = post.title;
    document.getElementById("content").value = post.content;
    document.getElementById("author").value = post.author;

    showMessage("Edit the details and click Save Post.");

    window.scrollTo({ top: 0, behavior: "smooth" });
}

function deletePost(id) {

    if (!confirm("Are you sure you want to delete this post?"))
        return;

    posts = posts.filter(post => post.id !== id);

    saveData();

    showResponse(
        200,
        "Post deleted successfully",
        null
    );

    showMessage("Post deleted successfully!");

    renderPosts();
}

function clearForm(clearMessage = true) {

    document.getElementById("postId").value = "";
    document.getElementById("title").value = "";
    document.getElementById("content").value = "";
    document.getElementById("author").value = "";

    if (clearMessage) {
        document.getElementById("message").textContent = "";
    }
}

renderPosts();

</script>

</body>
</html>
