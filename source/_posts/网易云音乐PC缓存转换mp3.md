---
title: 网易云音乐PC缓存转换mp3
date: 2020-08-16 22:42:40
tags: skills
menu: 
    Go Home: /index.html
categories: 技术分享
cover: /gallery/thumbnails/2020-8-16.jpg
thumbnail: /gallery/thumbnails/2020-8-16.jpg
toc: true
comments: true
---

# 1. 网易云音乐PC版缓存位置及编码方式

缓存文件位置：`C:\Users\当前用户名\AppData\Local\Netease\CloudMusic\Cache\Cache\`

缓存文件格式：`*.uc`

<!--more-->

参考资料可知，缓存文件与原始mp3文件的大小相同，且缓存文件中大量存在字符“A3”，故推测缓存文件的编码方式为**将原始mp3与0xa3异或得到**。

![010Editor](/gallery/pictures/2020-8-16/1.png)

# 2. 使用代码实现文件转换

## java版

```java
public class UI {
    private JFrame frame = new JFrame("转换器");
    private JPanel panel = new JPanel();
    private JLabel label1 = new JLabel("输入路径：");
    private JLabel label2 = new JLabel("输出路径：");
    private JLabel state = new JLabel();
    private JTextField inputPath = new JTextField();
    private JTextField outputPath = new JTextField();
    private JButton convert = new JButton("转换");
    private JButton cancel = new JButton("取消");
    private JButton chooseInputPath = new JButton("...");
    private JButton chooseOutputPath = new JButton("...");

    public UI(){
        frame.setSize(500, 200);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setLocationRelativeTo(null);

        label1.setBounds(10, 10, 80, 25);
        label2.setBounds(10, 40, 80, 25);
        inputPath.setBounds(90, 10, 350, 25);
        outputPath.setBounds(90, 40, 350, 25);
        chooseInputPath.setBounds(450, 10, 30, 25);
        chooseOutputPath.setBounds(450, 40, 30, 25);
        state.setBounds(10, 70, 200, 30);
        state.setFont(new Font("微软雅黑", Font.PLAIN, 13));
        state.setForeground(Color.PINK);

        convert.setBounds(10, 110, 100, 25);
        cancel.setBounds(300, 110, 100, 25);

        chooseInputPath.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                JFileChooser fileChooser = new JFileChooser();
                fileChooser.setCurrentDirectory(new File("."));
                fileChooser.setFileSelectionMode(JFileChooser.FILES_ONLY);
                fileChooser.setFileFilter(new FileNameExtensionFilter("*.uc", "uc"));
                int result = fileChooser.showOpenDialog(frame);
                if(result == JFileChooser.APPROVE_OPTION){
                    File file = fileChooser.getSelectedFile();
                    String str = file.getAbsolutePath();
                    inputPath.setText(str);
                    str = str.substring(0, str.length()-3);
                    outputPath.setText(str + ".mp4");
                }
            }
        });
        cancel.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                System.exit(0);
            }
        });
        convert.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                state.setText("正在转换，请稍后......");
                ucConvert(inputPath.getText(), outputPath.getText());
                state.setText("转换完成！");
            }
        });

        panel.setLayout(null);
        panel.add(label1);
        panel.add(label2);
        panel.add(inputPath);
        panel.add(outputPath);
        panel.add(chooseInputPath);
        panel.add(chooseOutputPath);
        panel.add(convert);
        panel.add(cancel);
        panel.add(state);

        frame.add(panel);
        frame.setVisible(true);
    }
    public void ucConvert(String inputPath, String outputPath){
        try{
            File inFile = new File(inputPath);
            File outFile = new File(outputPath);

            DataInputStream dis = new DataInputStream( new FileInputStream(inFile));
            DataOutputStream dos = new DataOutputStream( new FileOutputStream(outFile));
            byte[] by = new byte[1000];
            int len;
            while((len=dis.read(by))!=-1){
                System.out.println(len);
                for(int i=0;i<len;i++){
                    by[i]^=0xa3;
                }
                dos.write(by,0,len);
            }
            dis.close();
            dos.close();
        }catch(IOException ioe){
            System.err.println(ioe);
        }
    }
    
    public static void main(String[] args){
        new UI();
    }
}
```

由于带有GUI界面，代码较长，核心代码仅为ucConvert()函数部分。（转载）

## C#版

### Form1.Designer.cs

```c#
namespace CloudMusicDecode
{
    partial class Form1
    {
        /// <summary>
        /// 必需的设计器变量。
        /// </summary>
        private System.ComponentModel.IContainer components = null;

        /// <summary>
        /// 清理所有正在使用的资源。
        /// </summary>
        /// <param name="disposing">如果应释放托管资源，为 true；否则为 false。</param>
        protected override void Dispose(bool disposing)
        {
            if (disposing && (components != null))
            {
                components.Dispose();
            }
            base.Dispose(disposing);
        }

        #region Windows 窗体设计器生成的代码

        /// <summary>
        /// 设计器支持所需的方法 - 不要修改
        /// 使用代码编辑器修改此方法的内容。
        /// </summary>
        private void InitializeComponent()
        {
            this.groupBox1 = new System.Windows.Forms.GroupBox();
            this.chooseInputPath = new System.Windows.Forms.Button();
            this.inputPath = new System.Windows.Forms.TextBox();
            this.inputPathLabel = new System.Windows.Forms.Label();
            this.groupBox2 = new System.Windows.Forms.GroupBox();
            this.chooseOutPath = new System.Windows.Forms.Button();
            this.checkBox1 = new System.Windows.Forms.CheckBox();
            this.outPath = new System.Windows.Forms.TextBox();
            this.outPathLabel = new System.Windows.Forms.Label();
            this.startButton = new System.Windows.Forms.Button();
            this.closeButton = new System.Windows.Forms.Button();
            this.progressBar1 = new System.Windows.Forms.ProgressBar();
            this.groupBox1.SuspendLayout();
            this.groupBox2.SuspendLayout();
            this.SuspendLayout();
            // 
            // groupBox1
            // 
            this.groupBox1.Controls.Add(this.chooseInputPath);
            this.groupBox1.Controls.Add(this.inputPath);
            this.groupBox1.Controls.Add(this.inputPathLabel);
            this.groupBox1.Font = new System.Drawing.Font("微软雅黑", 9F, System.Drawing.FontStyle.Regular, System.Drawing.GraphicsUnit.Point, ((byte)(134)));
            this.groupBox1.Location = new System.Drawing.Point(12, 12);
            this.groupBox1.Name = "groupBox1";
            this.groupBox1.Size = new System.Drawing.Size(758, 100);
            this.groupBox1.TabIndex = 0;
            this.groupBox1.TabStop = false;
            this.groupBox1.Text = "输入";
            // 
            // chooseInputPath
            // 
            this.chooseInputPath.Location = new System.Drawing.Point(709, 41);
            this.chooseInputPath.Name = "chooseInputPath";
            this.chooseInputPath.Size = new System.Drawing.Size(30, 27);
            this.chooseInputPath.TabIndex = 2;
            this.chooseInputPath.Text = "...";
            this.chooseInputPath.UseVisualStyleBackColor = true;
            this.chooseInputPath.Click += new System.EventHandler(this.chooseInputPath_Click);
            // 
            // inputPath
            // 
            this.inputPath.Location = new System.Drawing.Point(82, 41);
            this.inputPath.Name = "inputPath";
            this.inputPath.Size = new System.Drawing.Size(620, 27);
            this.inputPath.TabIndex = 1;
            // 
            // inputPathLabel
            // 
            this.inputPathLabel.AutoSize = true;
            this.inputPathLabel.Location = new System.Drawing.Point(6, 47);
            this.inputPathLabel.Name = "inputPathLabel";
            this.inputPathLabel.Size = new System.Drawing.Size(69, 20);
            this.inputPathLabel.TabIndex = 0;
            this.inputPathLabel.Text = "文件路径";
            // 
            // groupBox2
            // 
            this.groupBox2.Controls.Add(this.chooseOutPath);
            this.groupBox2.Controls.Add(this.checkBox1);
            this.groupBox2.Controls.Add(this.outPath);
            this.groupBox2.Controls.Add(this.outPathLabel);
            this.groupBox2.Font = new System.Drawing.Font("微软雅黑", 9F, System.Drawing.FontStyle.Regular, System.Drawing.GraphicsUnit.Point, ((byte)(134)));
            this.groupBox2.Location = new System.Drawing.Point(12, 118);
            this.groupBox2.Name = "groupBox2";
            this.groupBox2.Size = new System.Drawing.Size(758, 125);
            this.groupBox2.TabIndex = 1;
            this.groupBox2.TabStop = false;
            this.groupBox2.Text = "输出";
            // 
            // chooseOutPath
            // 
            this.chooseOutPath.Location = new System.Drawing.Point(709, 44);
            this.chooseOutPath.Name = "chooseOutPath";
            this.chooseOutPath.Size = new System.Drawing.Size(30, 27);
            this.chooseOutPath.TabIndex = 3;
            this.chooseOutPath.Text = "...";
            this.chooseOutPath.UseVisualStyleBackColor = true;
            this.chooseOutPath.Click += new System.EventHandler(this.chooseOutPath_Click);
            // 
            // checkBox1
            // 
            this.checkBox1.AutoSize = true;
            this.checkBox1.Location = new System.Drawing.Point(82, 88);
            this.checkBox1.Name = "checkBox1";
            this.checkBox1.Size = new System.Drawing.Size(274, 24);
            this.checkBox1.TabIndex = 3;
            this.checkBox1.Text = "在缓存文件目录下创建同名mp4文件";
            this.checkBox1.UseVisualStyleBackColor = true;
            this.checkBox1.CheckedChanged += new System.EventHandler(this.checkBox1_CheckedChanged);
            // 
            // outPath
            // 
            this.outPath.Location = new System.Drawing.Point(82, 44);
            this.outPath.Name = "outPath";
            this.outPath.Size = new System.Drawing.Size(620, 27);
            this.outPath.TabIndex = 2;
            // 
            // outPathLabel
            // 
            this.outPathLabel.AutoSize = true;
            this.outPathLabel.Location = new System.Drawing.Point(6, 47);
            this.outPathLabel.Name = "outPathLabel";
            this.outPathLabel.Size = new System.Drawing.Size(69, 20);
            this.outPathLabel.TabIndex = 1;
            this.outPathLabel.Text = "输出路径";
            // 
            // startButton
            // 
            this.startButton.Font = new System.Drawing.Font("微软雅黑", 9F, System.Drawing.FontStyle.Regular, System.Drawing.GraphicsUnit.Point, ((byte)(134)));
            this.startButton.Location = new System.Drawing.Point(12, 249);
            this.startButton.Name = "startButton";
            this.startButton.Size = new System.Drawing.Size(90, 30);
            this.startButton.TabIndex = 2;
            this.startButton.Text = "开始转换";
            this.startButton.UseVisualStyleBackColor = true;
            this.startButton.Click += new System.EventHandler(this.startButton_Click);
            // 
            // closeButton
            // 
            this.closeButton.Font = new System.Drawing.Font("微软雅黑", 9F, System.Drawing.FontStyle.Regular, System.Drawing.GraphicsUnit.Point, ((byte)(134)));
            this.closeButton.Location = new System.Drawing.Point(680, 249);
            this.closeButton.Name = "closeButton";
            this.closeButton.Size = new System.Drawing.Size(90, 30);
            this.closeButton.TabIndex = 3;
            this.closeButton.Text = "退出";
            this.closeButton.UseVisualStyleBackColor = true;
            this.closeButton.Click += new System.EventHandler(this.closeButton_Click);
            // 
            // progressBar1
            // 
            this.progressBar1.Location = new System.Drawing.Point(109, 249);
            this.progressBar1.Name = "progressBar1";
            this.progressBar1.Size = new System.Drawing.Size(565, 29);
            this.progressBar1.TabIndex = 4;
            // 
            // Form1
            // 
            this.AutoScaleDimensions = new System.Drawing.SizeF(8F, 15F);
            this.AutoScaleMode = System.Windows.Forms.AutoScaleMode.Font;
            this.BackColor = System.Drawing.SystemColors.Control;
            this.ClientSize = new System.Drawing.Size(782, 303);
            this.Controls.Add(this.progressBar1);
            this.Controls.Add(this.closeButton);
            this.Controls.Add(this.startButton);
            this.Controls.Add(this.groupBox2);
            this.Controls.Add(this.groupBox1);
            this.Name = "Form1";
            this.Text = "网易云缓存转换";
            this.groupBox1.ResumeLayout(false);
            this.groupBox1.PerformLayout();
            this.groupBox2.ResumeLayout(false);
            this.groupBox2.PerformLayout();
            this.ResumeLayout(false);

        }

        #endregion

        private System.Windows.Forms.GroupBox groupBox1;
        private System.Windows.Forms.Button chooseInputPath;
        private System.Windows.Forms.TextBox inputPath;
        private System.Windows.Forms.Label inputPathLabel;
        private System.Windows.Forms.GroupBox groupBox2;
        private System.Windows.Forms.Button chooseOutPath;
        private System.Windows.Forms.CheckBox checkBox1;
        private System.Windows.Forms.TextBox outPath;
        private System.Windows.Forms.Label outPathLabel;
        private System.Windows.Forms.Button startButton;
        private System.Windows.Forms.Button closeButton;
        private System.Windows.Forms.ProgressBar progressBar1;
    }
}


```

### Form1.cs

```c#
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace CloudMusicDecode
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            InitializeComponent();
        }

        private void chooseInputPath_Click(object sender, EventArgs e)
        {
            OpenFileDialog fileDialog = new OpenFileDialog();
            fileDialog.Multiselect = false;
            fileDialog.Title = "选择文件";
            fileDialog.Filter = "uc缓存文件(*.uc)|*.uc";
            if(fileDialog.ShowDialog() == DialogResult.OK)
            {
                inputPath.Text = fileDialog.FileName;
            }
        }

        private void closeButton_Click(object sender, EventArgs e)
        {
            System.Environment.Exit(0);
        }

        private void chooseOutPath_Click(object sender, EventArgs e)
        {
            SaveFileDialog fileDialog = new SaveFileDialog();
            fileDialog.Title = "选择保存路径";
            fileDialog.Filter = "音频文件(*.mp4)|*.mp4";
            if(fileDialog.ShowDialog() == DialogResult.OK)
            {
                outPath.Text = fileDialog.FileName;
            }

        }

        private void checkBox1_CheckedChanged(object sender, EventArgs e)
        {
            if(checkBox1.Checked == true)
            {
                outPath.Enabled = false;
                if(inputPath.Text != "")
                {
                    int length = inputPath.Text.Length;
                    string str = inputPath.Text.Substring(0, length-3);
                    outPath.Text = str + ".mp4";
                }      
            }
            else
            {
                outPath.Enabled = true;
            }
        }

        private void startButton_Click(object sender, EventArgs e)
        {
            

            if (inputPath.Text == "" || outPath.Text == "")
            {
                MessageBox.Show("请选择正确的文件路径！", "错误", MessageBoxButtons.OK);
            }
            else
            {
                FileStream inputFile = new FileStream(inputPath.Text, FileMode.Open);
                FileStream outputFile = new FileStream(outPath.Text, FileMode.Create, FileAccess.Write);
                byte[] buf = new byte[1024];

                progressBar1.Visible = true;
                progressBar1.Minimum = 1;
                progressBar1.Maximum = (int)new FileInfo(inputPath.Text).Length;
                progressBar1.Step = 1024;

                while (true)
                {
                    int len = inputFile.Read(buf, 0, buf.Length);
                    if(len > 0)
                    {
                        for (int i = 0; i < len; i++)
                        {
                            buf[i] ^= 0xa3;
                        }
                        outputFile.Write(buf, 0, len);
                    }
                    else
                    {
                        break;
                    }
                    startProgress();
                }
                outputFile.Close();
                inputFile.Close();
            }
        }

        private void startProgress()
        {
            progressBar1.PerformStep();
        }
    }
}

```



效果展示：

![CloudMusicDecode](/gallery/pictures/2020-8-16/2.png)

下载地址：https://pan.baidu.com/s/1DLTKRo2yE95IpqzviZdMVQ

提取码：2btc

# 3. 其他

![？？？](/gallery/pictures/2020-8-16/3.png)

经过努力找到这样一个在线解析网址：

http://tool.liumingye.cn/music/?page=homePage

![支持音乐链接](/gallery/pictures/2020-8-16/4.png)

![支持VIP歌曲无损格式下载](/gallery/pictures/2020-8-16/5.png)

草...

> 一点疑惑：使用whois查询可知该站部署于阿里云服务器，所以为什么作者不怕律师函警告呢？

