---
layout: post
title: C#-winform控件多语言-json
date: 2022-06-14
categories: blog
tags: [C#,.net,json]
description: .net-Winform通过json实现控件多语言
---
# 问题描述
通过json实现winform窗体控件多语言

# 需要用到
Newtonsoft.Json

# 思路
- 解析json
- 遍历控件，修改text

# 解决方案
先对json进行解析
``` 
        private static string lang;
        private string source;
        private StreamReader file;
        private Dictionary<string, string> dict = new Dictionary<string, string>();
        public trans()
        {
            lang = IniReadValue(iniFilePath, "Language", "lang");
            logger.DebugFormat("Language:{0}", lang);
            try
            {
                file = new StreamReader(new FileStream("lang/" + lang + ".json", FileMode.Open, FileAccess.Read, FileShare.Read));
            }
            catch (IOException)
            {
                file = new StreamReader(new FileStream("lang/en_us.json", FileMode.Open, FileAccess.Read, FileShare.Read));
                General.logger.Error("Can't find settings.ini!");
            }
            catch (Exception)
            {
                General.logger.Fatal("Can't load language file!");
                ErrorForm form = new ErrorForm("Can't load language file!");
            }
            source = file.ReadToEnd();
            file.Close();
            source = source.Replace("\r", "").Replace("\n", "").Replace("\t", "");
            //source = file.ToString();
            JsonTextReader reader = new JsonTextReader(new StringReader(source));
            string tmp1 = "";
            string tmp2 = "";
            while (reader.Read())
            {
                if (reader.Value != null)
                {
                    //Console.WriteLine("Token: {0}, Value: {1}", reader.TokenType, reader.Value);
                    if (reader.TokenType == JsonToken.PropertyName)
                    {
                        tmp1 = reader.Value.ToString();
                    }
                    else if (reader.TokenType == JsonToken.String)
                    {
                        tmp2 = reader.Value.ToString();
                        dict[tmp1] = tmp2;
                        tmp1 = "";
                        tmp2 = "";
                    }
                }
                else
                {
                    //Console.WriteLine("Token: {0}", reader.TokenType);
                }
            }
        }
        ~trans()
        {
            file.Dispose();
        }
```

做一个return

```
        public string tr(string s)
        {
            try
            {
                return dict[s];
            }
            catch (Exception)
            {
                General.logger.WarnFormat("Translate string not found! Source:({0})", s);
                return s;
            }
        }
```

最后写一个函数对form的控件进行遍历
```
        public void Init(Form form)
        {
            foreach (Control control in form.Controls)
            {
                try
                {
                    control.Text = tr("Form.Control".Replace("Form", form.Name).Replace("Control", control.Name));
                }
                catch (Exception)
                {
                    //Actually we do nothing if we catch an exception.
                }
            }
        }
```