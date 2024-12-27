# Python based SQL dynamic translation scripting




        '''
        读取DB2数据表格，并转译为Excel
        @Author: J. X. PENG (Alreitetrs Pourton)
        @Date: 2024.02.02
        '''



        import openpyxl
        import re


        if __name__ == '__main__':



        # 使用 'r' 参数来读取文件
    try:
        # 提示用户输入文件名
        file_name = input("请输入文件名: ")

        # 尝试以读取模式打开文件，并指定编码为 Windows-1252
        with open(file_name, 'r', encoding='Windows-1252') as file:
            content = file.read()



        # 正则表达式提取
        # 开始位置
        start_marker = 'CREATE TABLE'
        # 结束位置
        end_marker = 'COMMENT ON '
        # 使用正则表达式找到起始和结束位置
        start_pos = re.search(start_marker, content).end()
        end_pos = re.search(end_marker, content[start_pos:]).start() + start_pos
        # 提取特定范围内的内容
        text = content[start_pos:end_pos]


        # 提取字段英文名
        # 使用正则表达式匹配双引号内的内容
        matches = re.findall(r'"([^"]*)"', text)


        # 提取字段类型
        # 提取第一个左括号的位置
        matchbracket = re.search(r'\(', content[start_pos:end_pos])
        # 定义 start_pos2
        start_pos2 = matchbracket.start() + start_pos
        # 提取特定范围内的内容
        text2 = content[start_pos2:end_pos]
        # 使用逗号分割文本
        split_text = text2.split(',')
        # 替换每个字符串中的"/n/t/t"为空白
        split_text2 = [item.replace("\n\t\t", " ") for item in split_text]
        # 去除每个字符串两端的空格
        split_text3 = [item.strip() for item in split_text2]
        # 对每个字符串使用空格分割，并取第一个空格后的字符串
        matches2 = [re.findall(r'\b[A-Za-z]+\b', item)[0] for item in split_text3]


        # 提取字段长度
        # 初始化一个空列表来存储结果
        matches3 = []
        # 首先，按逗号分隔文本字符串
        text_length = text2.split(',')
        # 使用for循环遍历列表中的每一项
        for item2 in text_length:
            # 对每一项应用正则表达式
            match3 = re.search(r'\((\d+)\)', item2)
            if match3:
                # 如果找到匹配项，将括号内的数字添加到结果列表中
                matches3.append(match3.group(1))
            else:
                # 如果没有找到匹配项，添加一个空字符串到结果列表中
                matches3.append('')



        # 正则表达式提取另一个范围
        start_marker2 = 'PRIMARY KEY'  # 开始位置
        end_marker2 = 'SET'  # 结束位置
        # 使用正则表达式找到起始和结束位置
        start_pos2 = re.search(start_marker2, content).end()
        end_pos2 = re.search(end_marker2, content[start_pos2:]).start() + start_pos2
        # 提取特定范围内的内容
        text_new = content[start_pos2:end_pos2]


        # 提取主键
        # 使用正则表达式匹配双引号内的内容
        matches4_ = re.findall(r'"([^"]*)"', text_new)
        # 去除最后两项(最后两项为文件名）
        matches4_ = matches4_[:-2]
        # 开始遍历从matches的第三项（索引为2）开始
        matches4 = ['true' if match in matches4_ else 'false' for match in matches[2:-2]]


        # 提取是否必填
        # 判断每一项是否有“NOT NULL"
        matches5 = ["true" if "NOT NULL" in item else "false" for item in split_text2]


        # 提取字段密级（均为“3”）
        matches6 = [3 for _ in matches[2:-2]]


        # 提取字段脱敏（均为“0”）
        matches7 = [0 for _ in matches[2:-2]]


        # 提取分区（均为“false”）
        matches8 = ["false" for _ in matches[2:-2]]



        # 写入Excel表格
        # 打开Excel文件
        workbook = openpyxl.load_workbook('/Users/alreiters/Desktop/hiveImportTemplate.xlsx')
        worksheet = workbook.active


        # 写入字段英文名
        # 写入前两行,因为这两项为文件名（注意：这里使用正确的行索引1和2）
        for i in range(1, 3):  # Excel的行索引从1开始
            worksheet.cell(row=i, column=1, value=matches[i - 1])  # 减1以匹配matches的索引
        # 从第三行开始写入到倒数第三行，因为最后两项为表名和索引名
        for idx, match in enumerate(matches[2:-2], start=10):  # 从第三项开始，并设置start=3以匹配Excel的行索引
            worksheet.cell(row=idx, column=1, value=match)


        # 写入字段类型
        for idx2, match2 in enumerate(matches2, start=10):
            worksheet.cell(row=idx2, column=3, value=match2)


        # 写入字段类型
        for idx3, match3 in enumerate(matches3, start=10):
            worksheet.cell(row=idx3, column=4, value=match3)


        # 写入主键
        for idx4, match4 in enumerate(matches4, start=10):
            worksheet.cell(row=idx4, column=5, value=match4)


        # 写入是否必填
        for idx5, match5 in enumerate(matches5, start=10):
            worksheet.cell(row=idx5, column=7, value=match5)


        # 写入字段密级
        for idx6, match6 in enumerate(matches6, start=10):
            worksheet.cell(row=idx6, column=8, value=match6)


        # 写入字段脱敏
        for idx7, match7 in enumerate(matches7, start=10):
            worksheet.cell(row=idx7, column=9, value=match7)


        # 写入分区
        for idx8, match8 in enumerate(matches8, start=10):
            worksheet.cell(row=idx8, column=12, value=match8)


        # 填写表头信息
        worksheet.cell(row=5, column=1, value=file_name)
        worksheet.cell(row=5, column=6, value=file_name)
        worksheet.cell(row=5, column=7, value=file_name)


        # 在最后一行填入分区信息
        # 判断表格有几项
        number = len(matches)
        # 减去头尾4项文件名，从第10行开始写入
        number = number-4+10
        # 写入各列固定信息
        worksheet.cell(row=number, column=1, value="dt")
        worksheet.cell(row=number, column=2, value="分区日期")
        worksheet.cell(row=number, column=3, value="string")
        worksheet.cell(row=number, column=4, value="10")
        worksheet.cell(row=number, column=5, value="true")
        worksheet.cell(row=number, column=7, value="true")
        worksheet.cell(row=number, column=12, value="true")
        worksheet.cell(row=number, column=13, value="1")


        # 另存为新的Excel表格文件
        workbook.save("/Users/alreiters/Desktop/hiveImportTemplate1.xlsx")



    #和一开始的“try”构成异常处理结构
    except FileNotFoundError:
        print("文件未找到，请检查文件路径是否正确。")
    except Exception as e:
        print(f"发生错误: {e}")
