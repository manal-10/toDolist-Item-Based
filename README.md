# toDolist-Item-Based
# doList.h
#ifndef DOLIST_H
#define DOLIST_H

#include <QMainWindow>
#include <QListWidget>
#include <QDir>

QT_BEGIN_NAMESPACE
namespace Ui { class todoApp; }
QT_END_NAMESPACE

class todoApp : public QMainWindow
{
    Q_OBJECT

public:
    todoApp(QWidget *parent = nullptr);
    ~todoApp();
    void saveContent(QString filename);
    void openContent(QString filename);
    void dropEvent(QDropEvent * event) override;

private slots:
    void on_exit_triggered();

    void on_todayTaskAct_triggered();

    void on_pendingTaskAct_triggered();

    void on_finishedTaskAct_triggered();

    void on_newTask_triggered();

    void on_Save_triggered();

    void on_Open_triggered();

    void editTaskSlot();

    void on_actionDelete_Task_triggered();


private:
    Ui::todoApp *ui;


    QString filename = QDir::homePath() + "/tasks.txt";;

};
#endif // DOLIST_H

  # doList.cpp

#include "doList.h"
#include "ui_todoapp.h"
#include <task.h>
#include <QFile>
#include <QTextStream>
#include <QFileDialog>
#include <QDir>
#include <QRegExp>
#include <QDropEvent>
#include <QDebug>

todoApp::todoApp(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::todoApp)
{
    ui->setupUi(this);



    openContent(this->filename);


    connect(ui->todayTask, &QListWidget::itemDoubleClicked, this, &todoApp::editTaskSlot);
    connect(ui->pendingTask, &QListWidget::itemDoubleClicked, this, &todoApp::editTaskSlot);
    connect(ui->finishedTask, &QListWidget::itemDoubleClicked, this, &todoApp::editTaskSlot);
}

todoApp::~todoApp()
{
    delete ui;
}



void todoApp::saveContent(QString filename){


    QFile file(filename);


    if(file.open(QFile::WriteOnly | QFile::Text))
    {

        QTextStream in(&file);

        for(int i=0; i<ui->todayTask->count(); i++)
        {
            in << ui->todayTask->item(i)->text();

            in << "\n";
        }


        in << "\n";

        for(int i=0; i<ui->pendingTask->count(); i++)
        {
            in << ui->pendingTask->item(i)->text();
            in << "\n";
        }

        in << "\n";

        for(int i=0; i<ui->finishedTask->count(); i++)
        {
            in << ui->finishedTask->item(i)->text();
            in << "\n";
        }
    }

    file.close();
}



void todoApp::openContent(QString filename){


    QFile file(filename);

    if(file.open(QFile::ReadOnly | QFile::Text))
    {

        QTextStream out(&file);

        while(!out.atEnd())
        {

            QString line = out.readLine();

            if(!line.isEmpty())
            {

                    auto splitLine = line.split("\t");
                    QString status = splitLine[3].remove(0,10);

                    QRegExp exp("(\\d{2})[/](\\d{2})[/](\\d{4})");
                    exp.indexIn(line);
                    auto dateString = exp.capturedTexts()[0].trimmed();

                     if(status == "No")
                     {

                            auto dateList = dateString.split("/");


                            auto date = QDate(dateList.at(2).toInt(), dateList.at(1).toInt(), dateList.at(0).toInt());


                            if(date == QDate::currentDate())
                                 ui->todayTask->addItem(line);
                            else if(date > QDate::currentDate())
                                 ui->pendingTask->addItem(line);
                            else
                                 ui->finishedTask->addItem(line);
                       }
                       else
                            ui->finishedTask->addItem(line);
           }
    }

      file.close();
    }
}



void todoApp::on_newTask_triggered()
{

    Task T;

    auto reply = T.exec();

    if(reply == Task::Accepted)
    {
        if(T.getStatus() == "No")
        {
            if(T.getDate() == QDate::currentDate())
                ui->todayTask->addItem(T.getTask());
            else if(T.getDate() > QDate::currentDate())
                ui->pendingTask->addItem(T.getTask());
            else
                ui->finishedTask->addItem(T.getTask());
        }
        else
            ui->finishedTask->addItem(T.getTask());
    }


    saveContent(this->filename);
}

void todoApp::on_Open_triggered()
{

    filename = QFileDialog::getOpenFileName();


    openContent(filename);
}

void todoApp::on_Save_triggered()
{

    filename = QFileDialog::getSaveFileName();


    saveContent(filename);

}

void todoApp::on_exit_triggered()
{
    QApplication::quit();
}


void todoApp::on_todayTaskAct_triggered()
{
    if(ui->todayTask->isVisible() && ui->todayLabel->isVisible())
    {
        ui->todayTask->setVisible(false);
        ui->todayLabel->setVisible(false);
    }
    else
    {
        ui->todayTask->setVisible(true);
        ui->todayLabel->setVisible(true);
    }
}

void todoApp::on_pendingTaskAct_triggered()
{
    if(ui->pendingTask->isVisible() && ui->pendingLabel->isVisible())
    {
        ui->pendingTask->setVisible(false);
        ui->pendingLabel->setVisible(false);
    }
    else
    {
        ui->pendingTask->setVisible(true);
        ui->pendingLabel->setVisible(true);
    }
}

void todoApp::on_finishedTaskAct_triggered()
{
    if(ui->finishedTask->isVisible() && ui->finishedLabel->isVisible())
    {
        ui->finishedTask->setVisible(false);
        ui->finishedLabel->setVisible(false);
    }
    else
    {
        ui->finishedTask->setVisible(true);
        ui->finishedLabel->setVisible(true);
    }
}


void todoApp::editTaskSlot(){

    QListWidget *listWidget = dynamic_cast<QListWidget*>(sender());

    if(listWidget)
    {

        Task T;


        auto item = listWidget->currentItem()->text();


        auto splitItem = item.split("\t");

        T.setDesc(splitItem[0].remove(0, 13));
        T.setDate(splitItem[1].remove(0,5));
        T.setTag(splitItem[2].remove(0,5));
        T.setStatus(splitItem[3].remove(0,10));


        auto reply = T.exec();

        if(reply == Task::Accepted)
        {
            if(T.getStatus() == "No")
            {
                    if(T.getDate() == QDate::currentDate())
                    {

                        listWidget->takeItem(listWidget->row(listWidget->currentItem()));


                        ui->todayTask->addItem(T.getTask());
                    }
                    else if(T.getDate() > QDate::currentDate())
                    {

                        listWidget->takeItem(listWidget->row(listWidget->currentItem()));


                        ui->pendingTask->addItem(T.getTask());
                    }
                    else
                    {

                        listWidget->takeItem(listWidget->row(listWidget->currentItem()));


                        ui->finishedTask->addItem(T.getTask());
                    }
            }
            else
            {

                listWidget->takeItem(listWidget->row(listWidget->currentItem()));


                ui->finishedTask->addItem(T.getTask());
            }

        }


    }


    saveContent(this->filename);
}


void todoApp::on_actionDelete_Task_triggered()
{

    QList<QListWidgetItem *> todayItems = ui->todayTask->selectedItems();


    if(!todayItems.isEmpty())
    {
        for(auto item : todayItems)
            ui->todayTask->takeItem(ui->todayTask->row(item))                ;
    }



    QList<QListWidgetItem *> pendingItems = ui->pendingTask->selectedItems();

    if(!pendingItems.isEmpty())
    {
        for(auto item :pendingItems)
            ui->pendingTask->takeItem(ui->pendingTask->row(item))                ;
    }



    QList<QListWidgetItem *> finishedItems = ui->finishedTask->selectedItems();


    if(!finishedItems.isEmpty())
    {
        for(auto item : finishedItems)
            ui->finishedTask->takeItem(ui->finishedTask->row(item))               ;
    }




    QFile file(this->filename);


    if(file.open(QFile::ReadOnly | QFile::Text))
    {

        QTextStream out(&file);

        while(!out.atEnd())
        {

            QString line = out.readLine();


            if(!todayItems.isEmpty())
            {
                for(int i=0; i<todayItems.size(); i++)
                {
                    if(line == todayItems[i]->text())
                    {
                        file.remove(line);
                    }

                }
            }


            if(!pendingItems.isEmpty())
            {
                for(int i=0; i<pendingItems.size(); i++)
                {
                    if(line == pendingItems[i]->text())
                    {
                        file.remove(line);
                    }

                }
            }


            if(!finishedItems.isEmpty())
            {
                for(int i=0; i<finishedItems.size(); i++)
                {
                    if(line == finishedItems[i]->text())
                    {
                        file.remove(line);
                    }

                }
            }

        }
    }

    file.close();

    saveContent(this->filename);
}


void todoApp::dropEvent(QDropEvent *event)
{
    if(event->isAccepted())
    {
        Task T;
        T.exec();
    }
}

  
# task.h

  #ifndef TASK_H
#define TASK_H

#include <QDialog>
#include <QDate>

namespace Ui {
class Task;
}

class Task : public QDialog
{
    Q_OBJECT

public:
    explicit Task(QWidget *parent = nullptr);
    ~Task();


    QString getDesc();
    QString getStatus();
    QString getTag();
    QDate getDate();
    QString getTask();

    //Setters
    void setDesc(QString);
    void setStatus(QString);
    void setTag(QString);
    void setDate(QString date);

private:
    Ui::Task *ui;
};

#endif // TASK_H

  
# task.cpp

  #include "task.h"
#include "ui_task.h"

Task::Task(QWidget *parent) :
    QDialog(parent),
    ui(new Ui::Task)
{
    ui->setupUi(this);


    this->setWindowTitle("New Task");


    ui->dateEdit->setDate(QDate::currentDate());

}

Task::~Task()
{
    delete ui;
}

QString Task::getDesc(){
    return ui->descLineEdit->text();
}

QString Task::getStatus(){

    if(ui->finished->isChecked())
        return "Yes";
    else
        return "No";

}

QString Task::getTag(){
    return ui->tag->currentText();
}

QDate Task::getDate(){
    return ui->dateEdit->date();
}

QString Task::getTask(){
    return "Description: " + Task::getDesc() + "\t" + "Due: " + ui->dateEdit->text() + "\t"
            + "Tag: " + Task::getTag() + "\t"  + "Finished: " + Task::getStatus();
}

void Task::setDesc(QString desc){
    ui->descLineEdit->setText(desc);
}

void Task::setTag(QString currText){
    ui->tag->setCurrentText(currText);
}

void Task::setDate(QString str){


    auto dateList = str.split("/");


    if(!dateList.isEmpty())
    {
        auto date = QDate(dateList.at(2).toInt(), dateList.at(1).toInt(), dateList.at(0).toInt());
        ui->dateEdit->setDate(date);
    }
}

void Task::setStatus(QString status){
    if(status == "Yes")
        ui->finished->setChecked(true);
    else
        ui->finished->setChecked(false);
}


  
# main.cpp
#include "todoList.h"

#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    todoApp w;
    w.show();
    return a.exec();
}
