---
title: "[Go] 飞书电子表格导出Excel文件"
subtitle: ""
date: 2022-05-12T11:10:03+08:00
lastmod: 2022-05-12T11:10:03+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

飞书电子表格网页版有导出Excel功能，但API文档没有此接口，使用excelize流式写入实现demo如下

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"

	"github.com/xuri/excelize/v2"
)

type RespToken struct {
	Code              int    `json:"code"`
	Expire            int    `json:"expire"`
	Msg               string `json:"msg"`
	TenantAccessToken string `json:"tenant_access_token"`
}

type RespSpreadsheets struct {
	Code int `json:"code"`
	Data struct {
		Revision         int    `json:"revision"`
		Spreadsheettoken string `json:"spreadsheetToken"`
		Valuerange       struct {
			Majordimension string          `json:"majorDimension"`
			Range          string          `json:"range"`
			Revision       int             `json:"revision"`
			Values         [][]interface{} `json:"values"`
		} `json:"valueRange"`
	} `json:"data"`
	Msg string `json:"msg"`
}

// 获取 tenant_access_token（企业自建应用）
// https://open.feishu.cn/document/ukTMukTMukTM/ukDNz4SO0MjL5QzM/auth-v3/auth/tenant_access_token_internal
func get_tenant_access_token(app_id, app_secret string) (token string) {
	url := "https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal"

	payload := map[string]string{
		"app_id":     app_id,
		"app_secret": app_secret,
	}

	bytesData, _ := json.Marshal(payload)

	request, err := http.NewRequest("POST", url, bytes.NewBuffer(bytesData))
	if err != nil {
		panic(err)
	}

	request.Header.Set("Content-Type", "application/json; charset=utf-8")

	client := http.Client{}
	resp, err := client.Do(request)
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()

	statusCode := resp.StatusCode
	if statusCode == 200 {
		body, _ := ioutil.ReadAll(resp.Body)

		var Resp RespToken
		json.Unmarshal(body, &Resp)

		switch Resp.Code {
		case 0:
			token = Resp.TenantAccessToken
		}
	}

	return token
}

// 导出电子表格
// https://open.feishu.cn/document/ukTMukTMukTM/ugTMzUjL4EzM14COxMTN
func getSpreadsheets(token, spreadsheetToken, sheetId string) {
	url := "https://open.feishu.cn/open-apis/sheets/v2/spreadsheets/" + spreadsheetToken + "/values/" + sheetId
	request, err := http.NewRequest("GET", url, nil)
	if err != nil {
		panic(err)
	}

	request.Header.Set("Content-Type", "application/json; charset=utf-8")
	request.Header.Set("Authorization", "Bearer "+token)

	client := http.Client{}
	resp, err := client.Do(request)
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()

	statusCode := resp.StatusCode
	if statusCode == 200 {
		body, _ := ioutil.ReadAll(resp.Body)

		var Resp RespSpreadsheets
		json.Unmarshal(body, &Resp)
		values := Resp.Data.Valuerange.Values

		newExcel := excelize.NewFile()
		newExcel.Path = "./" + sheetId + ".xlsx"

		sheetName := newExcel.GetSheetName(newExcel.GetActiveSheetIndex())
		streamSheet, err := newExcel.NewStreamWriter(sheetName)
		if err != nil {
			panic(err)
		}

		for i, v := range values {
			// i从0开始，而excel是从第1行开始
			streamSheet.SetRow(fmt.Sprintf("A%d", i+1), v)
		}

		// 刷新至文件，此步骤不能缺
		if err := streamSheet.Flush(); err != nil {
			fmt.Println(err)
		}

		if err := newExcel.Save(); err != nil {
			fmt.Println(err)
		}
	}
}

func main() {
	app_id := "飞书应用app_id"
	app_secret := "飞书应用app_secret"
	spreadsheetToken := "飞书电子表格Token"
	sheetId := "飞书表格id"

	token := get_tenant_access_token(app_id, app_secret)
	getSpreadsheets(token, spreadsheetToken, sheetId)
}
```
