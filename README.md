<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Personalized Avatar Generator</title>
</head>
<body>
    <h1>Welcome to the Personalized Avatar Generator!</h1>
    <form action="/submit" method="POST">
        <label for="hair_color">Hair Color:</label>
        <select name="hair_color" id="hair_color">
            <option value="black">Black</option>
            <option value="brown">Brown</option>
            <option value="blonde">Blonde</option>
            <option value="red">Red</option>
        </select><br><br>

        <label for="eye_color">Eye Color:</label>
        <select name="eye_color" id="eye_color">
            <option value="brown">Brown</option>
            <option value="blue">Blue</option>
            <option value="green">Green</option>
            <option value="gray">Gray</option>
        </select><br><br>

        <label for="skin_color">Skin Color:</label>
        <select name="skin_color" id="skin_color">
            <option value="dark">Dark</option>
            <option value="medium">Medium</option>
            <option value="light">Light</option>
        </select><br><br>

        <label for="height">Height:</label>
        <select name="height" id="height">
            <option value="short">Short</option>
            <option value="average">Average</option>
            <option value="tall">Tall</option>
        </select><br><br>

        <label for="body_type">Body Type:</label>
        <select name="body_type" id="body_type">
            <option value="slim">Slim</option>
            <option value="standard">Standard</option>
            <option value="curvy">Curvy</option>
        </select><br><br>

        <button type="submit">Generate Avatar</button>
    </form>
</body>
</html>
