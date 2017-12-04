package main

import (
	"./checker"
	"encoding/json"
	"fmt"
	"gopkg.in/fatih/set.v0"
	"io"
	"io/ioutil"
	"log"
	"net/http"
	"os"
	"reflect"
	"strconv"
	"strings"
	"sync"
)

type FileTree struct {
	Name string
	Id   string
}

type Projectnod struct {
	Id                  int
	Name_with_namespace string
}

var (
	str_projectid              string
	files_tree                 []FileTree
	gitlab_url                 string
	list_file_url              string
	token_project_input        string
	project_input              int
	file_select_project_input  string
	user_project_input_file_id []string
	indexlist                  []int
)

var counter = 1 //计数器
var base_url = ""   //API基础地址

func get_projects_id() string {

	fmt.Println("请输入你的private_token:\n")
	fmt.Scanln(&token_project_input)
	gitlab_url = base_url + "?private_token=" + token_project_input
	resp, err := http.Get(gitlab_url)
	if err != nil {
		log.Fatalln(err)
	}
	defer resp.Body.Close()

	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {

		log.Fatalln(err)
	}
	var projects []Projectnod
	err = json.Unmarshal(body, &projects)
	if err != nil {
		fmt.Println("***************请检查token是否正确****************")
		log.Fatalln(err)
	}

	fmt.Println("选择项目，输入对应的序号\n")
START:
	for _, v := range projects {
		fmt.Println(counter, "     **     "+v.Name_with_namespace)

		indexlist = append(indexlist, counter)
		counter++
	}

	fmt.Scanln(&project_input)

	check_result, err := checker.Contain(project_input, indexlist)

	if !check_result && err != nil {
		fmt.Println("你所选择的序号没有对应的项目，请重新选择\n")
		counter = 1
		indexlist = indexlist[:0]
		goto START
	} else {
		projectid := projects[project_input-1].Id
		str_projectid = fmt.Sprintf("%d", projectid)
		return str_projectid

	}

}

func get_files_id() []FileTree {

	list_file_url = base_url + "/" + str_projectid + "/repository/tree?private_token=" + token_project_input
	resp, err := http.Get(list_file_url)
	if err != nil {
		// handle error
		log.Fatalln(err)
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)

	if err != nil {
		log.Fatalln(err)
	}

	err = json.Unmarshal(body, &files_tree)
	if err != nil {
		fmt.Println("提示:", "该项目没有任何可下载文件，程序将退出")
		log.Fatalln(err)
	}
	return files_tree

}

func download_files() /*(*simplejson.Json)*/ {
	counter = 1
	indexlist = indexlist[:0]
	var wg sync.WaitGroup

	fmt.Println("请选择你需要下载的文件序号,使用逗号隔开\n")

	for _, file := range files_tree {
		fmt.Println(counter, "     "+file.Name)
		indexlist = append(indexlist, counter)
		counter++

	}

	fmt.Scanln(&file_select_project_input)

	user_project_input_file_id = strings.Split(file_select_project_input, ",")

	s := set.New()
	t := set.New()

	for _, indexid := range indexlist {
		fmt.Println(reflect.TypeOf(indexid))
		s.Add(indexid)
	}

	for _, user_file_id := range user_project_input_file_id {
		int_user_file_id, err := strconv.Atoi(user_file_id)
		if err != nil {
			fmt.Println("输入错误，请使用逗号隔开，末尾不要带逗号")
			log.Fatalln(err)
		}
		t.Add(int_user_file_id)
		fmt.Println(reflect.TypeOf(int_user_file_id))
	}

	fmt.Println(s)
	fmt.Println(t)
	fmt.Println(s.IsSubset(t))

	if !s.IsSubset(t) {
		fmt.Println("输入错误，程序将退出，正群输入例如:1,2,3")
		log.Fatalln("exit")
	}

	for _, file_id := range user_project_input_file_id {
		wg.Add(1)
		int_file_id, err := strconv.Atoi(file_id)
		if err != nil {
			log.Fatalln(err)
		}

		go func() {
			defer wg.Done()
			download_url := base_url + "/" + str_projectid + "/repository/raw_blobs/" + files_tree[int_file_id-1].Id + "?private_token=" + token_project_input
			fmt.Println(download_url)
			filename := files_tree[int_file_id-1].Name

			out, err := os.Create(filename)
			if err != nil {
				log.Fatalln(err)
			}
			defer out.Close()

			resp, err := http.Get(download_url)
			if err != nil {
				log.Fatalln(err)
			}
			defer resp.Body.Close()

			_, err = io.Copy(out, resp.Body)
			if err != nil {
				log.Fatalln(err)
			}
		}()
	}
	wg.Wait()

}

func main() {
	get_projects_id()
	get_files_id()
	download_files()
	fmt.Println("下载完成，请查看目录。。。")

}

