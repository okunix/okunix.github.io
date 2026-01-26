+++
date = '{{ .Date }}'
draft = true
title = '{{ replace .File.ContentBaseName "-" " " | title }}'
slug = '{{ lower .File.ContentBaseName }}'
tags = []
authors = [ "Yusuf Yakubov" ]
+++
