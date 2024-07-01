#服务器 #alist #xiaoya

## 粘贴到==终端运行==
```
docker exec -i xiaoya sqlite3 data/data.db <<EOF
select value from x_setting_items where key = "token";
EOF
```

## 示例
![[Screenshot_2024-06-30-22-58-19-94_84d3000e3f4017145260f7618db1d683.jpg]]