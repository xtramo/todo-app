<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"> <!-- This tells the browser to use the right characters, like letters and symbols -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0"> <!-- This makes the page look good on phones and computers -->
  <title>To-Do App</title> <!-- The name that shows up on the browser tab -->
  <style>
    body { font-family: Arial, sans-serif; background: #f0f0f0; display: flex; justify-content: center; padding-top: 20px; }
    .container { background: white; padding: 20px; border-radius: 5px; box-shadow: 0 0 10px rgba(0,0,0,0.1); width: 300px; }
    .input-group { display: flex; margin-bottom: 10px; }
    input[type="text"] { flex: 1; padding: 5px; border: 1px solid #ccc; border-right: none; border-radius: 3px 0 0 3px; }
    button { padding: 5px 10px; background: #4CAF50; color: white; border: none; border-radius: 0 3px 3px 0; cursor: pointer; }
    button:hover { background: #45a049; }
    select, .toggle { width: 100%; padding: 5px; margin-bottom: 10px; border: 1px solid #ccc; border-radius: 3px; }
    ul { list-style: none; padding: 0; }
    li { display: flex; align-items: center; margin: 5px 0; }
    li.completed span { text-decoration: line-through; color: #888; }
    input[type="checkbox"] { margin-right: 10px; }
    .translation { font-size: 0.8em; color: #666; margin-left: 20px; display: none; }
    .translation.show { display: inline; }
  </style>
</head>
<body>
  <div class="container">
    <h1>To-Do App</h1> <!-- Big title at the top of our app -->
    <div class="input-group">
      <input type="text" id="taskInput" placeholder="Add a new task"> <!-- Where you type your task -->
      <button onclick="addTask()">Add</button> <!-- Button to add your task to the list -->
    </div>
    <select id="languageSelect" onchange="translateTasks()">
      <option value="en">English</option> <!-- Option to keep text in English -->
      <option value="es">Spanish</option> <!-- Option to translate to Spanish -->
      <option value="fr">French</option> <!-- Option to translate to French -->
      <option value="de">German</option> <!-- Option to translate to German -->
    </select>
    <label class="toggle">
      <input type="checkbox" id="showTranslation" onchange="toggleTranslation()">
      Show Translations <!-- Checkbox to show or hide translations -->
    </label>
    <ul id="taskList"></ul> <!-- The list where all your tasks will appear -->
  </div>

  <script>
    let tasks = []; // This is where we store all our tasks, like a notebook

    function addTask() {
      const input = document.getElementById('taskInput'); // Grab the text box where you type
      const text = input.value.trim(); // Get what you typed and remove extra spaces
      if (text) { // Only do this if you typed something
        tasks.push({ text, completed: false, translation: text }); // Add your task to the list
        input.value = ''; // Clear the text box
        translateTasks(); // Translate the new task
        renderTasks(); // Show the updated list
      }
    }

    function toggleTask(index) {
      tasks[index].completed = !tasks[index].completed; // Flip the "done" status of the task
      renderTasks(); // Update the list to show the change
    }

    async function getTranslation(text, targetLang) {
      try {
        const response = await fetch(`https://api.mymemory.translated.net/get?q=${encodeURIComponent(text)}&langpair=en|${targetLang}`); // Ask the internet to translate the text
        const data = await response.json(); // Get the translated text back
        return data.responseData.translatedText || `${text} (Translation failed)`; // Return the translation or the original if it fails
      } catch (error) {
        console.error('Translation error:', error); // Show an error if something goes wrong
        return `${text} (Error)`; // Use the original text if the translation fails
      }
    }

    async function translateTasks() {
      console.log('Translating tasks to', document.getElementById('languageSelect').value); // Let us know which language we're using
      const lang = document.getElementById('languageSelect').value; // Get the chosen language
      for (let task of tasks) {
        task.translation = await getTranslation(task.text, lang); // Translate each task
      }
      renderTasks(); // Update the list with new translations
    }

    function toggleTranslation() {
      const show = document.getElementById('showTranslation').checked; // See if the checkbox is checked
      console.log('Toggle translation:', show); // Tell us if translations are on or off
      const translations = document.getElementsByClassName('translation'); // Find all translation texts
      for (let t of translations) {
        t.classList.toggle('show', show); // Show or hide the translations
      }
    }

    function renderTasks() {
      const list = document.getElementById('taskList'); // Get the task list area
      list.innerHTML = ''; // Clear the old list
      tasks.forEach((task, index) => { // Go through each task
        const li = document.createElement('li'); // Make a new line for each task
        const checkbox = document.createElement('input'); // Create a checkbox
        checkbox.type = 'checkbox';
        checkbox.checked = task.completed; // Set if the task is done
        checkbox.onclick = () => toggleTask(index); // Let clicking the checkbox mark it done
        const span = document.createElement('span'); // Create a spot for the task text
        span.textContent = task.text; // Put the task name here
        if (task.completed) span.classList.add('completed'); // Cross it out if done
        const translation = document.createElement('span'); // Create a spot for the translation
        translation.textContent = task.translation; // Put the translated text here
        translation.classList.add('translation'); // Style it as a translation
        li.appendChild(checkbox); // Add the checkbox to the line
        li.appendChild(span); // Add the task text
        li.appendChild(translation); // Add the translation
        list.appendChild(li); // Add the whole line to the list
      });
    }

    document.getElementById('taskInput').addEventListener('keypress', function(e) { // Listen for when you press a key
      if (e.key === 'Enter') addTask(); // Add the task if you press Enter
    });

    // Start with everything in English
    translateTasks();
  </script>
</body>
</html>
