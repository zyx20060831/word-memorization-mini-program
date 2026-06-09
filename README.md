1.如何将doc文件转成txt，
首先小学和高中可以用一套方法转，打开wps，选择另存为，保存类型为txt，编码选择utf8（

初中词汇有老旧字体，需要先转成pdf，这样pdf会强制将老旧字体转换，接下来再转回文本文件，为了统一，选择utf8字体

为了处理utf8字体的txt，我们需要把vscode
环境也弄成utf8

处理数据文件的第一步是先去掉一些无用行，只保留单词行

接下来我们把不是单词和汉语的部分去掉
这样只剩下汉语和英语单词就比较好分类了

通过第一个汉字的位置，判断怎么切割


使用utf8字体不得不做出牺牲，第一个是判断是否是汉字时utf8和gbk的逻辑是不同的，第二个是easyx只收gbk，不收utf8，需要我们代码做出如图片一样的适应性调整




2.
原数据文件的中文名对vscode有点不太好用，需要我们自己把数据文件名称改为英文，否则读取会失败



反复开关绘画窗口，easyx会卡死，最好用完暂时把它隐藏起来



被注释掉的代码是个失败的提取数据的例子，事实上它对边缘情况处理得不够详尽，而且提取方法太过简单粗暴，直接把汉字部分和非汉字部分隔开

每个控制台都需要判断是否用户进行了正确选择，用choice_success做判断是否终止循环


UTF8有特殊符号，需要特殊关注，尤其是特殊空格和换行符
3.代码
#include <conio.h>
#include <windows.h>
#include<iostream>
#include<string>
#include<map>
#include<fstream>
#include<cstdlib>
#include<ctime>
#include<graphics.h>
enum RANGE{
    primary,junior,senior,errorbook
};
using namespace std;

bool graph_opened=false;

//utf8适配,识别utf8汉字用的
bool is_utf8_chinese_start(const std::string& s, int i){
    if(i + 2 >= (int)s.size()) return false;
    unsigned char c1 = (unsigned char)s[i];
    unsigned char c2 = (unsigned char)s[i+1];
    unsigned char c3 = (unsigned char)s[i+2];

    // CJK Unified Ideographs 常见范围：E4~E9
    if(c1 >= 0xE4 && c1 <= 0xE9 &&
       c2 >= 0x80 && c2 <= 0xBF &&
       c3 >= 0x80 && c3 <= 0xBF) return true;

    return false;
}
//由于easyx不收utf8，先转成gbk
string UTF8ToGBK(const string& utf8) {
    int wlen = MultiByteToWideChar(CP_UTF8, 0, utf8.c_str(), -1, NULL, 0);
    if (wlen <= 0) return utf8;

    wchar_t* wstr = new wchar_t[wlen];
    MultiByteToWideChar(CP_UTF8, 0, utf8.c_str(), -1, wstr, wlen);

    int len = WideCharToMultiByte(CP_ACP, 0, wstr, -1, NULL, 0, NULL, NULL);
    if (len <= 0) {
        delete[] wstr;
        return utf8;
    }

    char* gbk = new char[len];
    WideCharToMultiByte(CP_ACP, 0, wstr, -1, gbk, len, NULL, NULL);

    string result(gbk);

    delete[] wstr;
    delete[] gbk;

    return result;
}
//再来个函数嵌套
void outtextxy_utf8(int x, int y, const string& s) {
    string gbk = UTF8ToGBK(s);
    outtextxy(x, y, gbk.c_str());
}

void user_register();
void user_log_in();
void user_choose(string ID);
void user_practise(string ID,bool order,bool C_to_E,RANGE range);
void user_errorbook(string ID,bool order,bool C_to_E);
int main(){
    SetConsoleCP(65001);
    SetConsoleOutputCP(65001);//把控制台改成utf8

    srand((unsigned int)time(0));
    while(1){
        system("cls");
        bool is_exit=false;
        cout<<"请选择一个选项"<<endl;
        cout<<"1:登录   ";
        cout<<"2:注册   ";
        cout<<"3:退出"<<endl;
        char choice_user;
        cin>>choice_user;

        switch(choice_user){
            case '1':user_log_in();break;
            case '2':user_register();break;
            case '3':is_exit=true; break;
            default:cout<<"请输入正确的字符"<<endl;system("pause");
        }
        if(is_exit){
            break;
        }
    }

    if(graph_opened){
        closegraph();
    }

    return 0;
}
void user_register(){
    string ID;
    string password;
    string check_ID;
    string check_password;
    while(1){
        system("cls");
        cout<<"请输入账号名"<<endl;
        cin>>ID;
        ifstream search_ID("user.txt");
        if(!search_ID){
            ofstream temp("user.txt");
            temp.close();
            search_ID.clear();
            search_ID.open("user.txt");
        }
        bool able_continue=true;
        while(search_ID>>check_ID>>check_password){
            if(check_ID==ID){
                cout<<"账号已重复"<<endl;
                system("pause");
                able_continue=false;
                break;
            }
        }
        search_ID.close();
        if(able_continue){
            cout<<"请输入密码"<<endl;
            cin>>password;
            ofstream record_information("user.txt",ios::app);
            record_information<<ID<<" "<<password<<endl;
            record_information.close();
            user_choose(ID);
            return;
        }

    }
}
void user_log_in(){
    string ID;
    string password;
    string check_ID;
    string check_password;
    while(1){
        system("cls");
        cout<<"请输入账号名"<<endl;
        cin>>ID;
        cout<<"请输入密码"<<endl;
        cin>>password;
        ifstream search_information("user.txt");
        if(!search_information){
            ofstream temp("user.txt");
            temp.close();
            cout<<"账号不存在"<<endl;
            system("pause");
            return;
        }
        bool is_success=false;
        while(search_information>>check_ID>>check_password){
            if(check_ID==ID&&check_password==password){
                is_success=true;
                break;
            }
        }
        search_information.close();
        if(is_success){
            user_choose(ID);
            return;
        }
        else{
            cout<<"账号或密码错误"<<endl;
            system("pause");
        }
    }
}
void user_choose(string ID){
    char choice_user;
    bool order;bool C_to_E;RANGE range;
    while(1){
        system("cls");
        bool choice_success=true;
        cout<<"是否按顺序"<<endl;
        cout<<"1:是  2:否"<<endl;
        cin>>choice_user;
        switch(choice_user){
            case '1':order=true;break;
            case '2':order=false;break;
            default:choice_success=false;break;
        }
        if(choice_success){
            break;
        }
        cout<<"请输入规定字符"<<endl;
        system("pause");
    }    
    while(1){
        system("cls");
        bool choice_success=true;
        cout<<"英译汉还是汉译英"<<endl;
        cout<<"1:汉译英  2:英译汉"<<endl;
        cin>>choice_user;
        switch(choice_user){
            case '1':C_to_E=true;break;
            case '2':C_to_E=false;break;
            default:choice_success=false;break;
        }
        if(choice_success){
            break;
        }
        cout<<"请输入规定字符"<<endl;
        system("pause");
    }    
    while(1){
        system("cls");
        bool choice_success=true;
        cout<<"请选择范围"<<endl;
        cout<<"1:小学  2:初中  3:高中 4:错题本"<<endl;
        cin>>choice_user;
        switch(choice_user){
            case '1':range=primary;break;
            case '2':range=junior;break;
            case '3':range=senior;break;
            case '4':user_errorbook(ID,order,C_to_E);return;break;
            default:choice_success=false;break;
        }
        if(choice_success){
            break;
        }
        else{
        cout<<"请输入规定字符"<<endl;
        system("pause");
        }
    }

    

    user_practise(ID,order,C_to_E,range);

     

       

}

void user_practise(string ID,bool order,bool C_to_E,RANGE range){

    //将词汇表存入二维数组
    string filename;
    if(range==primary)filename="primary.txt";
    else if(range==junior)filename="junior.txt";
    else filename="senior.txt";
   

    ifstream readin(filename.c_str());
 

    
    string word[5000][2];
    int word_num=0;
    string line;
    int line_no=0;

    while(getline(readin,line)){
        line_no++;

        //每个文件按固定行号硬排除
        if(range==primary){
            if(line_no==1||line_no==2||line_no==23||line_no==37||line_no==48||
               line_no==80||line_no==133||line_no==165||line_no==179||
               line_no==191||line_no==214||line_no==228||line_no==281||
               line_no==318||line_no==329||line_no==347||line_no==355||
               line_no==364||line_no==377||line_no==382||line_no==389||
               line_no==397||line_no==438||line_no==499||line_no==508||
               line_no==521||(line_no>=398&&line_no<=416)){
                continue;
            }

            
            if(line_no==307) line="TV room 电视房";
            if(line_no==487) line="cute/kju:t/可爱的";
        }
        else if(range==junior){
            if(line_no<=19||line_no==105||line_no==107||line_no==855||
               line_no==856||line_no==1556||line_no==1669||line_no==1670){
                continue;
            }

            //1555和1556合并成一行
            if(line_no==1555){
                line="1532  program(programme) n 程序,项目,节目";
            }
        }
        else if(range==senior){
            if(line_no==1||line_no==2||line_no==316||line_no==571||
               line_no==940||line_no==1141||line_no==1293||line_no==1487||
               line_no==1593||line_no==1732||line_no==1827||line_no==1862||
               line_no==1889||line_no==2006||line_no==2190||line_no==2278||
               line_no==2373||line_no==2664||line_no==2681||line_no==2874||
               line_no==3363||line_no==3599||line_no==3667||line_no==3721||
               line_no==3874||line_no==3876||line_no==3894||line_no>=3904){
                continue;
            }
        }

        // 去掉特殊空格，尤其是utf8的特殊符号
        size_t pos;
        ;
        while((pos=line.find("\xC2\xA0"))!=string::npos) line.replace(pos,2," ");
         while((pos=line.find("\xE3\x80\x80"))!=string::npos) line.replace(pos,3," ");
        
        while((pos=line.find("\r"))!=string::npos)line.erase(pos,1);

        while(!line.empty()&&(line[0]==' '||line[0]=='\t')) line.erase(0,1);//去掉开头符号
        while(!line.empty()&&(line[line.size()-1]==' '||line[line.size()-1]=='\t')) line.erase(line.size()-1,1);
        //去掉结尾符号1
        if(line.empty()) continue;

        //初中、高中去掉前面的编号
        if(range==junior||range==senior){
            int i=0;
            while(i<(int)line.size()&&line[i]>='0'&&line[i]<='9') i++;
            while(i<(int)line.size()&&(line[i]==' '||line[i]=='.'||line[i]=='-'||line[i]=='\t')) i++;
            if(i>0) line=line.substr(i);
        }
        //再清一遍开头结尾
        while(!line.empty()&&(line[0]==' '||line[0]=='\t')) line.erase(0,1);
        while(!line.empty()&&(line[line.size()-1]==' '||line[line.size()-1]=='\t')) line.erase(line.size()-1,1);
        if(line.empty()) continue;

        bool has_english=false;
        int first_chinese=-1;

        for(int j=0;j<(int)line.size();j++){
            if((line[j]>='a'&&line[j]<='z')||(line[j]>='A'&&line[j]<='Z')){
                has_english=true;
            }
            if(first_chinese==-1&&is_utf8_chinese_start(line,j)){
                first_chinese=j;
            }
        }

        if(!has_english||first_chinese==-1) continue;
  
        string english_part=line.substr(0,first_chinese);
        string chinese_part=line.substr(first_chinese);

        //英文去音标
        pos=english_part.find("[");
        if(pos!=string::npos) english_part=english_part.substr(0,pos);
        pos=english_part.find("/");
        if(pos!=string::npos) english_part=english_part.substr(0,pos);

        //英文去词性
        string tags[]={
            " v.aux"," adj."," adv."," prep."," conj."," pron."," phr.",
            " num."," art."," int."," vt."," vi."," n."," v."," a."," ad.",
            " adj"," adv"," prep"," conj"," pron"," phr"," num"," art",
            " int"," vt"," vi"," n"," v"," a"," ad"
        };
        for(int t=0;t<31;t++){
            pos=english_part.find(tags[t]);
            if(pos!=string::npos){
                english_part=english_part.substr(0,pos);
            }
        }
       //分别对汉字和英语部分清头清尾
        while(!english_part.empty()&&(english_part[0]==' '||english_part[0]=='\t')) english_part.erase(0,1);
        while(!english_part.empty()&&(english_part[english_part.size()-1]==' '||english_part[english_part.size()-1]=='\t')) english_part.erase(english_part.size()-1,1);

        while(!chinese_part.empty()&&(chinese_part[0]==' '||chinese_part[0]=='\t')) chinese_part.erase(0,1);
        while(!chinese_part.empty()&&(chinese_part[chinese_part.size()-1]==' '||chinese_part[chinese_part.size()-1]=='\t')) chinese_part.erase(chinese_part.size()-1,1);

        if(english_part.empty()||chinese_part.empty()) continue;

        word[word_num][0]=english_part;
        word[word_num][1]=chinese_part;
        word_num++;
    }
   

        readin.close();
 
/*/  //对前面数字之类删去一些
        int i=0;
        while(i<line.size()&&line[i]>='0'&&line[i]<='9')i++;
        while(i<line.size()&&(line[i]==' '||line[i]=='.'||line[i]=='-'))i++;
        line=line.substr(i);
        
        bool has_english=false;
        for(int j=0;j<(int)line.size();j++){
            if((line[j]>='a'&&line[j]<='z')||(line[j]>='A'&&line[j]<='Z')){
            has_english=true;
             }
            
            }
        
       

        
        
        //第一个中文位置
        int first_chinese=-1;
       
        for(int j=0;j<line.size();j++){
            if(is_utf8_chinese_start(line,j)){
                first_chinese=j;
                break;
            }
        }

        //看看此词是不是既有中文，又有英文
        if(!has_english||first_chinese==-1)continue;
        //if来处理特殊情况
        
        if(line.find("缩写形式")!=-1||line.find("复数形式")!=-1){
            int p=line.find(']');
            if(p!=-1){
                word[word_num][0]=line.substr(0,p+1);
                word[word_num][1]=line.substr(p+1);
            }
            //else 就是正常的把中文字符和非中文字符隔开
            else{
                word[word_num][0]=line.substr(0,first_chinese);
                word[word_num][1]=line.substr(first_chinese);
            }
        }
        else{
            word[word_num][0]=line.substr(0,first_chinese);
            word[word_num][1]=line.substr(first_chinese);
        }

        word_num++;
    }
    readin.close();/*/
    //画布
    
    if(!graph_opened){
        initgraph(800,800);
        graph_opened=true;
    }
    else{
        ShowWindow(GetHWnd(), SW_SHOW); // 防止easyx崩溃
    }
    
    

    setbkcolor(WHITE);
    settextcolor(BLACK);
    setbkmode(TRANSPARENT);
    settextstyle(28,0,"宋体");
    


    int array=0;
    bool is_quit=false;
    while(1){
        //先编号号
        int question;
        if(order){
            question=array;
            array++;
            if(array>=word_num){
                array=0;
            }
        }
        else{
            question=rand()%word_num;
        }
       
        string choice[4];
        for(int i=0;i<4;i++){
            int temp=rand()%word_num;
            if(C_to_E)choice[i]=word[temp][0];
            else choice[i]=word[temp][1];
        }
        int answer_place=rand()%4;
        if(C_to_E)choice[answer_place]=word[question][0];
        else choice[answer_place]=word[question][1];


       //展示UI
        cleardevice();
        setfillcolor(BLACK);
        solidrectangle(0,200,800,202);
        solidrectangle(0,400,800,402);
        solidrectangle(0,600,800,602);
        solidrectangle(400,200,402,600);
        rectangle(100,650,300,730);
        rectangle(500,650,700,730);
        if(C_to_E){
            outtextxy_utf8(40,60,"请选择对应的英文：");
            outtextxy_utf8(40,110,word[question][1].c_str());
        }
        else{
            outtextxy_utf8(40,60,"请选择对应的中文：");
            outtextxy_utf8(40,110,word[question][0].c_str());
        }

        outtextxy_utf8(40,270,(choice[0]).c_str());
        outtextxy_utf8(440,270,(choice[1]).c_str());
        outtextxy_utf8(40,470,(choice[2]).c_str());
        outtextxy_utf8(440,470,(choice[3]).c_str());
        outtextxy_utf8(560,680,"退出");
        while(1){
            ExMessage msg;
            if(peekmessage(&msg,EX_MOUSE)){
                if(msg.message==WM_LBUTTONDOWN){
                    int x=msg.x;
                    int y=msg.y;

                    if(x>=500&&x<=700&&y>=650&&y<=730){
                        is_quit=true;
                        break;
                    }

                    int user_answer=-1;
                    if(x>=0&&x<=400&&y>=200&&y<=400)user_answer=0;
                    else if(x>=400&&x<=800&&y>=200&&y<=400)user_answer=1;
                    else if(x>=0&&x<=400&&y>=400&&y<=600)user_answer=2;
                    else if(x>=400&&x<=800&&y>=400&&y<=600)user_answer=3;

                    if(user_answer!=-1){
                        if(user_answer==answer_place){
                            outtextxy_utf8(330,680,"回答正确");
                            Sleep(800);
                            break;
                        }
                        else{
                            outtextxy_utf8(330,620,"回答错误");
                            outtextxy_utf8(330,660,"正确答案：");
                            outtextxy_utf8(480,660,choice[answer_place].c_str());

                            rectangle(80,710,280,780);
                            rectangle(310,710,510,780);
                            rectangle(540,710,740,780);

                            outtextxy_utf8(110,735,"加入错题本");
                            outtextxy_utf8(370,735,"下一题");
                            outtextxy_utf8(610,735,"退出");

                            while(1){
                                ExMessage msg2;
                                if(peekmessage(&msg2,EX_MOUSE)){
                                    if(msg2.message==WM_LBUTTONDOWN){
                                        int x2=msg2.x;
                                        int y2=msg2.y;

                                        if(x2>=80&&x2<=280&&y2>=710&&y2<=780){
                                            ofstream fout((ID+"_errorbook.txt").c_str(),ios::app);
                                            fout<<word[question][0]<<"|"<<word[question][1]<<endl;
                                            fout.close();
                                            outtextxy_utf8(330,735,"已加入");
                                            Sleep(200);
                                            break;
                                        }

                                        if(x2>=310&&x2<=510&&y2>=710&&y2<=780){
                                            break;
                                        }

                                        if(x2>=540&&x2<=740&&y2>=710&&y2<=780){
                                            is_quit=true;
                                            break;
                                        }
                                    }
                                }

                                if(is_quit){
                                    break;
                                }

                                Sleep(10);
                            }
                            break;
                        }
                    }
                }
            }

            if(is_quit){
                break;
            }

            Sleep(10);
        }
        if(is_quit){
            break;
        }
    }
    ShowWindow(GetHWnd(), SW_HIDE); // 防止easyx崩溃

}

void user_errorbook(string ID,bool order,bool C_to_E){
    //把错题本存二维数组
    string word[5000][2];
    int word_num=0;
    string line;
//下面是防止错题本错题数目不够四个时的备用词汇
    string prepare[10][2]={
        {"apple","苹果"},
        {"book","书"},
        {"school","学校"},
        {"teacher","老师"},
        {"student","学生"},
        {"water","水"},
        {"friend","朋友"},
        {"family","家庭"},
        {"help","帮助"},
        {"answer","回答"}
    };

    ifstream readin((ID+"_errorbook.txt").c_str());

    while(getline(readin,line)){
        int p=line.find("|");
        if(p==-1)continue;

        word[word_num][0]=line.substr(0,p);
        word[word_num][1]=line.substr(p+1);
        word_num++;
    }

    readin.close();

    if(word_num==0){
        cout<<"错题本为空"<<endl;
        system("pause");
        return;
    }
//画布
   
    if(!graph_opened){
        initgraph(800,800);
        graph_opened=true;
    }
    else{
        ShowWindow(GetHWnd(), SW_SHOW); // 防止easyx崩溃
    }
    
    setbkcolor(WHITE);
    settextcolor(BLACK);
    setbkmode(TRANSPARENT);
    settextstyle(28,0,"宋体");

    int array=0;
    bool is_quit=false;
    while(1){
        int question;
        if(order){
            question=array;
            array++;
            if(array>=word_num){
                array=0;
            }
        }
        else{
            question=rand()%word_num;
        }

        string choice[4];

        for(int i=0;i<4;i++){
            if(word_num>=4){
                int temp=rand()%word_num;
                if(C_to_E)choice[i]=word[temp][0];
                else choice[i]=word[temp][1];
            }
            else{
                int temp=rand()%10;
                if(C_to_E)choice[i]=prepare[temp][0];
                else choice[i]=prepare[temp][1];
            }
        }

        int answer_place=rand()%4;
        if(C_to_E)choice[answer_place]=word[question][0];
        else choice[answer_place]=word[question][1];

        cleardevice();
        setfillcolor(BLACK);
        solidrectangle(0,200,800,202);
        solidrectangle(0,400,800,402);
        solidrectangle(0,600,800,602);
        solidrectangle(400,200,402,600);
        rectangle(500,650,700,730);

        if(C_to_E){
            outtextxy_utf8(40,60,"请选择对应的英文：");
            outtextxy_utf8(40,110,word[question][1].c_str());
        }
        else{
            outtextxy_utf8(40,60,"请选择对应的中文：");
            outtextxy_utf8(40,110,word[question][0].c_str());
        }

        outtextxy_utf8(40,270,(choice[0]).c_str());
        outtextxy_utf8(440,270,(choice[1]).c_str());
        outtextxy_utf8(40,470,(choice[2]).c_str());
        outtextxy_utf8(440,470,(choice[3]).c_str());
        outtextxy_utf8(560,680,"退出");

        while(1){
            ExMessage msg;
            if(peekmessage(&msg,EX_MOUSE)){
                if(msg.message==WM_LBUTTONDOWN){
                    int x=msg.x;
                    int y=msg.y;

                    if(x>=500&&x<=700&&y>=650&&y<=730){
                        is_quit=true;
                        break;
                    }

                    int user_answer=-1;
                    if(x>=0&&x<=400&&y>=200&&y<=400)user_answer=0;
                    else if(x>=400&&x<=800&&y>=200&&y<=400)user_answer=1;
                    else if(x>=0&&x<=400&&y>=400&&y<=600)user_answer=2;
                    else if(x>=400&&x<=800&&y>=400&&y<=600)user_answer=3;

                    if(user_answer!=-1){
                        if(user_answer==answer_place){
                            outtextxy_utf8(330,680,"回答正确");
                            Sleep(800);
                            break;
                        }
                        else{
                            outtextxy_utf8(330,620,"回答错误");
                            outtextxy_utf8(330,660,"正确答案：");
                            outtextxy_utf8(480,660,choice[answer_place].c_str());
                            Sleep(800);
                            break;
                        }
                    }
                }
            }

            if(is_quit){
                break;
            }

            Sleep(10);
        }
        if(is_quit){
            break;
        }
    }
    ShowWindow(GetHWnd(), SW_HIDE); // 防止easyx崩溃

}

4.
