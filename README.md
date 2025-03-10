# lunarsc

using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Drawing.Drawing2D;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using Newtonsoft.Json;
using XulusAPI;

namespace Lunar
{
    public partial class Lunar : Form
    {

        private string scriptsFolder = Path.Combine(Application.StartupPath, "scripts");
        private string savedFilePath = Path.Combine(Application.StartupPath, "selectedFile.txt");
        public Lunar()
        {
            InitializeComponent();
            LoadSavedText();
            // Optional: Set the form's border style to none if you want to fully control the border
            this.FormBorderStyle = FormBorderStyle.None;
        }

        // Override the OnPaint method to add rounded corners
        protected override void OnPaint(PaintEventArgs e)
        {
            base.OnPaint(e);

            // Create a rounded rectangle path
            int radius = 30; // Adjust radius for corner curvature
            GraphicsPath path = new GraphicsPath();
            path.AddArc(0, 0, radius, radius, 180, 90); // Top-left corner
            path.AddArc(this.Width - radius - 1, 0, radius, radius, 270, 90); // Top-right corner
            path.AddArc(this.Width - radius - 1, this.Height - radius - 1, radius, radius, 0, 90); // Bottom-right corner
            path.AddArc(0, this.Height - radius - 1, radius, radius, 90, 90); // Bottom-left corner
            path.CloseAllFigures();

            // Set the form's Region to the rounded rectangle
            this.Region = new Region(path);
        }

        // Optional: Add custom title bar drag functionality (if you're using FormBorderStyle.None)
        protected override void OnMouseDown(MouseEventArgs e)
        {
            if (e.Button == MouseButtons.Left)
            {
                // Corrected: Now calling methods directly
                ReleaseCapture();
                SendMessage(this.Handle, 0x112, 0xF012, 0); // Move window message
            }
            base.OnMouseDown(e);
        }

        // Import necessary functions for moving the form
        [System.Runtime.InteropServices.DllImport("user32.dll")]
        private static extern bool ReleaseCapture();

        [System.Runtime.InteropServices.DllImport("user32.dll")]
        private static extern IntPtr SendMessage(IntPtr hwnd, uint msg, uint wparam, uint lparam);

        private void Form1_Load(object sender, EventArgs e)
        {

        }

        private void LunarLogo_Click(object sender, EventArgs e)
        {

        }

        private void LunarName_TextChanged(object sender, EventArgs e)
        {

        }

        private void InjectBtn_Click(object sender, EventArgs e)
        {
            Api.Inject();
        }

        private void ExecuteBtn_Click(object sender, EventArgs e)
        {
            Api.Execute(Editor.Text);
        }

        private void ExitBtn_Click(object sender, EventArgs e)
        {
            this.Close();
        }

        private void TabOffBtn_Click(object sender, EventArgs e)
        {
            this.WindowState = FormWindowState.Minimized;
        }

        private void Editor_TextChanged(object sender, EventArgs e)
        {
            RichTextBox editor = sender as RichTextBox;
            if (editor == null) return;

            int selectionStart = editor.SelectionStart;
            string text = editor.Text;

            // Update LineCounter RichTextBox
            RichTextBox lineCounter = editor.Parent.Controls["LineCounter"] as RichTextBox;
            if (lineCounter != null)
            {
                lineCounter.Clear();
                for (int i = 1; i <= editor.Lines.Length; i++)
                {
                    lineCounter.AppendText(i + "\n");
                }
            }

            HashSet<string> luaKeywords = new HashSet<string>
    {
        "print", "humanoid", "workspace", "game", "script", "local", "function", "end",
        "if", "then", "else", "elseif", "for", "while", "repeat", "until", "return",
        "loadstring"
    };

            // Set text size to be 3x bigger
            editor.Font = new System.Drawing.Font("Consolas", 19);

            // Reset all text to white first
            editor.SelectAll();
            editor.SelectionColor = System.Drawing.Color.White;

            foreach (var keyword in luaKeywords)
            {
                int index = 0;
                while ((index = text.IndexOf(keyword, index)) != -1)
                {
                    editor.Select(index, keyword.Length);
                    editor.SelectionColor = System.Drawing.Color.Blue;
                    index += keyword.Length;
                }
            }

            // Highlight strings ("text")
            int quoteIndex = 0;
            while ((quoteIndex = text.IndexOf('"', quoteIndex)) != -1)
            {
                int endQuoteIndex = text.IndexOf('"', quoteIndex + 1);
                if (endQuoteIndex != -1)
                {
                    editor.Select(quoteIndex, endQuoteIndex - quoteIndex + 1);
                    editor.SelectionColor = System.Drawing.Color.Green;
                    quoteIndex = endQuoteIndex + 1;
                }
                else
                {
                    break;
                }
            }

            // Highlight dots (.)
            int dotIndex = 0;
            while ((dotIndex = text.IndexOf('.', dotIndex)) != -1)
            {
                editor.Select(dotIndex, 1);
                editor.SelectionColor = System.Drawing.Color.Red;
                dotIndex++;
            }

            // Highlight parentheses ()
            int parenIndex = 0;
            while ((parenIndex = text.IndexOf('(', parenIndex)) != -1)
            {
                editor.Select(parenIndex, 1);
                editor.SelectionColor = System.Drawing.Color.Purple;
                parenIndex++;
            }
            parenIndex = 0;
            while ((parenIndex = text.IndexOf(')', parenIndex)) != -1)
            {
                editor.Select(parenIndex, 1);
                editor.SelectionColor = System.Drawing.Color.Purple;
                parenIndex++;
            }

            // Highlight URLs
            string[] urls = { "https://raw.githubusercontent.com/", "https://pastebin.com/raw/" };
            foreach (var url in urls)
            {
                int urlIndex = 0;
                while ((urlIndex = text.IndexOf(url, urlIndex)) != -1)
                {
                    editor.Select(urlIndex, url.Length);
                    editor.SelectionColor = System.Drawing.Color.Orange;
                    urlIndex += url.Length;
                }
            }

            editor.Select(selectionStart, 0);
            editor.SelectionColor = System.Drawing.Color.Black;
        }


        private void label1_Click(object sender, EventArgs e)
        {

        }

        private void Tabs_TextChanged(object sender, EventArgs e)
        {

        }


        private void Script1_Click(object sender, EventArgs e)
        {
            using (OpenFileDialog openFileDialog = new OpenFileDialog())
            {
                openFileDialog.Filter = "Text and Lua files (*.txt;*.lua)|*.txt;*.lua";
                openFileDialog.Title = "Select a .txt or .lua file";

                if (openFileDialog.ShowDialog() == DialogResult.OK)
                {
                    string selectedFile = openFileDialog.FileName;
                    string fileName = Path.GetFileName(selectedFile);
                    string destinationPath = Path.Combine(scriptsFolder, fileName);

                    if (!Directory.Exists(scriptsFolder))
                    {
                        Directory.CreateDirectory(scriptsFolder);
                    }

                    // Copy the file to the scripts folder
                    File.Copy(selectedFile, destinationPath, true);

                    // Save the file path for persistence
                    File.WriteAllText(savedFilePath, destinationPath);

                    // Update button text and load content
                    Script1.Text = fileName;
                    Editor.Text = File.ReadAllText(destinationPath);
                }
            }
        }


        private void LoadSavedFile()
        {
            if (File.Exists(savedFilePath))
            {
                string savedPath = File.ReadAllText(savedFilePath);

                if (File.Exists(savedPath))
                {
                    Script1.Text = Path.GetFileName(savedPath);
                    Editor.Text = File.ReadAllText(savedPath);
                }
            }
        }

        private void LoadSavedText()
        {
            if (File.Exists(savedFilePath))
            {
                string savedPath = File.ReadAllText(savedFilePath);

                if (File.Exists(savedPath))
                {
                    Script1.Text = Path.GetFileName(savedPath);
                    Editor.Text = File.ReadAllText(savedPath);
                }
            }
        }

        private void JoinDiscordBtn_Click(object sender, EventArgs e)
        {
            string url = "https://discord.gg/getlunarr";
            System.Diagnostics.Process.Start(new System.Diagnostics.ProcessStartInfo(url) { UseShellExecute = true });
        }

        private void EndRobloxBtn_Click(object sender, EventArgs e)
        {
            Api.KillRoblox();
        }

        private void ScriptHubBtn_Click(object sender, EventArgs e)
        {
            // Create and show the ScriptHub form
            ScriptHub scriptHubForm = new ScriptHub();
            scriptHubForm.Show(); // Use ShowDialog() if you want it to be modal
        }
    }
}
