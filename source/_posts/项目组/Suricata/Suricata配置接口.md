---
title: Suricata配置接口
tags:
  - - Suricata
  - - API
categories:
  - - 项目组
    - Suricata
abbrlink: ad19baf
date: 2021-08-18 14:22:16
---
> 为`Suricata`对应的WEB页面开发编写相关接口用以实现配置修改与规则简单管理。

## 库引入、接口声明、常量定义
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <dirent.h>

static char **read_rules_file_name();
static void alter_select_rules(char **rules_list, int count);
static void alter_HOME_NET(char **HOME_NET_list, int count);
static void alter_mpm_algo(char *mpm_algo);
static void alter_spm_algo(char *spm_algo);
static void alter_rule_action(char *rules_name, char *sid, char *action);
static void add_custom_rule(char *action, char *protocol, char *stream_rule, char *option_rule);
static void insert_specific_line_to_file(char *ins_content, char *load_file, char *des_opt);
static char *read_rule_sid(char *src_rule);
static void strrp(char *src, char *sub, char *rp, char *p);

#define RULES_FILES_PATH "/var/lib/suricata/rules"
#define CUSTOMRULES_FILE_PATH "/var/lib/suricata/rules/custom.rules"
#define CONFIG_YAML_FILR_PATH "/etc/suricata/suricata.yaml"

char *names[100];
char sid[20];
```
## 接口定义
```c

/*
 * 功能：读取 RULES_FILES_PATH 路径下所有规则文件名
 * 传参：无
 * 返回：names规则名二维数组
 * usage：
 * read_rules_file_name();
 */
static char **read_rules_file_name()
{
    int n = 0;
    DIR *directory_pointer;
    struct dirent *entry;
    char *suffix;

    if ((directory_pointer = opendir(RULES_FILES_PATH)) == NULL)
        printf("Error opening \n ");
    else
    {
        while ((entry = readdir(directory_pointer)) != NULL)
        {
            suffix = strrchr(entry->d_name, '.');
            if (suffix != NULL && strcmp(suffix, ".rules") == 0)
            {
                names[n++] = entry->d_name;
            }
        }
        closedir(directory_pointer);
    }
    return names;
}

/*
 * 功能：将选择启用的规则集文件写入 CONFIG_YAML_FILR_PATH
 * 传参：rules_list规则集名称列表,count共启用规则集数目
 * 返回：无
 * usage:
 * char *a[] = {"ftp.rules","dns.rules","dos.rules"};
 * alter_select_rules(a,3);
 */
static void alter_select_rules(char **rules_list, int count)
{
    char rule_files[1000] = {"rule-files: ["};
    for (int i = 0; i < count; i++)
    {
        if (i != 0)
        {
            strcat(rule_files, ",");
        }
        strcat(rule_files, rules_list[i]);
    }
    strcat(rule_files, "]\n");
    insert_specific_line_to_file(rule_files, CONFIG_YAML_FILR_PATH, "rule-files:");
}

/*
 * 功能：将选择启用的规则集文件写入 CONFIG_YAML_FILR_PATH
 * 传参：rules_list规则集名称列表,count共启用规则集数目
 * 返回：无
 * usage:
 * char *a[] = {"192.168.0.0/16","10.0.0.0/8","172.16.0.0/12"};
 * alter_select_rules(a,3);

 */
static void alter_HOME_NET(char **HOME_NET_list, int count)
{
    char HOME_NET[1000] = {"    HOME_NET: \"["};
    for (int i = 0; i < count; i++)
    {
        if (i != 0)
        {
            strcat(HOME_NET, ",");
        }
        strcat(HOME_NET, HOME_NET_list[i]);
    }
    strcat(HOME_NET, "]\"\n");
    insert_specific_line_to_file(HOME_NET, CONFIG_YAML_FILR_PATH, "HOME_NET:");
}

/*
 * 功能：修改配置文件CONFIG_YAML_FILR_PATH中的mpm_algo选项
 * 传参：mpm_algo待修改值
 * 返回：无
 * usage:
 * alter_mpm_algo("hs");
 */
static void alter_mpm_algo(char *mpm_algo)
{
    char mpm_algo_option[1000] = {"mpm-algo: "};
    strcat(mpm_algo_option, mpm_algo);
    strcat(mpm_algo_option, "\n");
    insert_specific_line_to_file(mpm_algo_option, CONFIG_YAML_FILR_PATH, "mpm-algo:");
}

/*
 * 功能：修改配置文件CONFIG_YAML_FILR_PATH中的spm_algo选项
 * 传参：spm_algo待修改值
 * 返回：无
 * usage:
 * alter_spm_algo("hc");
 */
static void alter_spm_algo(char *spm_algo)
{
    char spm_algo_option[1000] = {"spm-algo: "};
    strcat(spm_algo_option, spm_algo);
    strcat(spm_algo_option, "\n");
    insert_specific_line_to_file(spm_algo_option, CONFIG_YAML_FILR_PATH, "spm-algo:");
}

/*
 * 功能：修改指定规则的动作
 * 传参：rules_name规则集名称，sid规则标识符，action动作名称
 * 返回：无
 * usage:
 * alter_rule_action("ftp.rules", "2026691", "alert");
 */
static void alter_rule_action(char *rules_name, char *sid, char *action)
{
    FILE *fp;
    FILE *fc;
    char *temp_file = (char *)malloc(strlen(RULES_FILES_PATH) + strlen("temp.rules") + 1);
    sprintf(temp_file, "%s/%s", RULES_FILES_PATH, "temp.rules");
    int line_hit = 0;
    int ch;
    char line[1000];
    char temp_line[1000];
    char *actions[] = {"alert", "pass", "drop", "reject"};
    char *load_file = (char *)malloc(strlen(RULES_FILES_PATH) + strlen(rules_name) + 1);
    sprintf(load_file, "%s/%s", RULES_FILES_PATH, rules_name);

    fc = fopen(temp_file, "w");
    fp = fopen(load_file, "r");

    if (fp == NULL || fc == NULL)
    {
        printf("Error opening \n ");
    }
    else
    {
        while (!feof(fp) && !line_hit)
        {
            fgets(line, 1000, fp);
            if (strstr(line, sid))
            {
                line_hit = 1;
            }
            else
            {
                fprintf(fc, "%s", line);
            }
        }

        strrp(line, actions[0], action, temp_line);
        memset(line, 0, 1000);
        strrp(temp_line, actions[1], action, line);
        memset(temp_line, 0, 1000);
        strrp(line, actions[2], action, temp_line);
        memset(line, 0, 1000);
        strrp(temp_line, actions[3], action, line);
        fprintf(fc, "%s", line);
        while ((ch = fgetc(fp)) != EOF)
        {
            fprintf(fc, "%c", ch);
        }

        fclose(fc);
        fclose(fp);
        remove(load_file);
        rename(temp_file, load_file);
    }
}

/*
 * 功能：添加自定义规则到 CUSTOMRULES_FILE_PATH 文件中
 * 传参：action动作名称，protocol协议名称，stream_rule流量方向规则，option_rule选项规则
 * 返回：无
 * usage:
 * char a[1000] = "msg:"ET MALWARE STOLENPENCIL CnC Domain in DNS Lookup"; dns.query; content:"client-message.com"; nocase; fast_pattern; endswith; reference:url,asert.arbornetworks.com/stolen-pencil-campaign-targets-academia/; classtype:command-and-control; sid:2026691; rev:4; metadata:affected_product Windows_XP_Vista_7_8_10_Server_32_64_Bit, attack_target Client_Endpoint, created_at 2018_12_05, deployment Perimeter, former_category MALWARE, malware_family StolenPencil, performance_impact Low, signature_severity Major, updated_at 2020_09_16;";
 * add_custom_rule("alert", "dns", "$HOME_NET any -> any any", a);
 */
static void add_custom_rule(char *action, char *protocol, char *stream_rule, char *option_rule)
{
    char *rule_content = (char *)malloc(strlen(action) + strlen(protocol) + strlen(stream_rule) + strlen(option_rule) + 20);
    sprintf(rule_content, "%s %s %s (%s)", action, protocol, stream_rule, option_rule);

    FILE *fp = fopen(CUSTOMRULES_FILE_PATH, "a+");
    if (fp == NULL)
    {
        printf("Error opening \n ");
    }
    else
    {
        fseek(fp, 0, SEEK_END);
        fwrite(rule_content, strlen(rule_content), 1, fp);
        fclose(fp);
    }
}

/*
 * 功能：在文本中将指定内容插入在某一字段所在行(覆盖该行方式)
 * 传参：line待插入的整行内容，load_file待插入的文件名，des_opt被覆盖行的特征字段
 * 返回：无
 * usage:
 * char a[1000] = "rule-file: [ftp.rules,dns.rules]";
 * insert_specific_line_to_file(a,CONFIG_YAML_FILR_PATH,"rule-file:");
 */
static void insert_specific_line_to_file(char *ins_content, char *load_file, char *des_opt)
{
    FILE *fp;
    FILE *fc;
    int line_hit = 0;
    int ch;
    char line[1000];
    char temp_file[100] = {};
    strcat(temp_file, load_file);
    strcat(temp_file, ".temp");

    fp = fopen(load_file, "r");
    fc = fopen(temp_file, "w");

    if (fp == NULL || fc == NULL)
    {
        printf("Error opening \n ");
    }
    else
    {
        while (!feof(fp) && !line_hit)
        {
            fgets(line, 1000, fp);
            if (strstr(line, des_opt))
            {
                line_hit = 1;
            }
            else
            {
                fprintf(fc, "%s", line);
            }
        }
        fprintf(fc, "%s", ins_content);
        while ((ch = fgetc(fp)) != EOF)
        {
            fprintf(fc, "%c", ch);
        }

        fclose(fc);
        fclose(fp);
        remove(load_file);
        rename(temp_file, load_file);
    }
}

/*
 * 功能：sid读取
 * 传参：src_rule源规则字符串
 * 返回：sid规则标识符
 * usage:
 * char a[1000] = "alert dns $HOME_NET any -> any any (msg:"ET MALWARE STOLENPEN...... sid:2026691; ......ajor, updated_at 2020_09_16;)";
 * char b[10] = read_rule_sid(a);
 */
static char *read_rule_sid(char *src_rule)
{
    int n = 0;
    char *index = strstr(src_rule, "sid:") + 4;
    int position = index - src_rule;

    while (src_rule[position] != ';')
    {
        sid[n++] = src_rule[position++];
    }
    return sid;
}

/*
 * 功能：字符串替换
 * 传参：src源字符串，sub待替换内容，rp替换内容，p输出字符串
 * 返回：无
 * char line[1000] = "aabbccddxxyyzz";
 * char temp_line[1000];
 * char action[10] = "drop";
 * char rep[10] = "alert";
 * strrp(line, actions, rep, temp_line);
 */
static void strrp(char *src, char *sub, char *rp, char *p)
{
    int sub_len = strlen(sub);
    char *po = NULL, *q = src;

    while ((po = strstr(q, sub)) != NULL)
    {
        strncat(p, q, po - q);
        strcat(p, rp);
        q += po - q + sub_len;
    }
    strcat(p, q);
}
```
