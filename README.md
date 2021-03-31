# using-Google-Forms-quizzes-with-a-testbank

Here's a quick and dirty way to use the new Google Forms Quizzes with a test bank of your own creation. 

To do this, you'll be running Google Apps Scripts. 

https://developers.google.com/apps-script/

There is a lot of information out there on how to use it, but basically you write a script and save it to Google Drive. You will do the editing and running using 

https://script.google.com

You'll also need to create 3 files (I show you examples of each below)...

* Your test bank (Google Sheets)
* A Google Forms Quiz template (Google Forms)
* A form to pick which questions you want to pull from your test bank for a quiz (Google Sheets)

...and make a copy of my Google Apps Script, modifying it to work with your files. 

Here's some examples of each, formatted in such a way that they will work with the Google Apps Script that I wrote. 

-----
**Test bank**

https://docs.google.com/spreadsheets/d/185vtr8aD-p4NCa5-QyfRJNSba2a1X_yjkkPFF9w66jk/edit?usp=sharing

Note the formatting. This type of formatting is hard coded in the script, but you can see what I'm getting at. 

I've bolded the correct answers, but this is not used when the quiz is created. It's just for the instructor's guidance and you can choose not to do this. 

For "Type", "MC" is multiple choice and "CB" is checkbox, the only two options that Google Quizzes currently takes. 

You can also store images by URL ("Type" = "IU") or stored on your Drive and referenced by ID ("Type" = "ID"). 

-----
**Google Forms Quiz template**

https://docs.google.com/forms/d/1ww-rNPzzr6Jdn3NyqKfo3CqfDofaWGhygFU6BTHyhBk/edit?usp=sharing

Google Apps Scripts does not have features for the Quizzes options, so my workaround is just to make a copy of a quiz template. 

-----
**Quiz maker**

https://docs.google.com/spreadsheets/d/10jBngMHdglLk8TcDGOUTEMbfFCxh1ywAFQ1_E3oxwVQ/edit?usp=sharing

This is the spreadsheet you'll use to make each quiz. You use this to pick the questions you want to use from the test bank and set the title of the quiz. 

-----

Once you have your own versions of these, you'll make a copy of this script

https://script.google.com/d/1rACPT_W6E2u9taMKEIf2IijP6onTUtAyurojSB5di3MCd4deZsgS3KZK/edit?usp=sharing

and edit three places in the beginning, putting in the ID's of your versions of the forms described above. 

From the script...
```javascript
function make_a_new_quiz() {

  /////////////////////////////////////////////////////////////
  // This should be the only things you need to edit for your own use
  /////////////////////////////////////////////////////////////

  // These define the ID that you need for your own Google Drive documents. 
  // The ID is a unique identifier for each file. You can grab it from the URL
  // for the document. For instance, if the URL reads...
  //
  //    https://script.google.com/a/siena.edu/d/1rACPT_W6E2u9taMKEIf2IijP6onTUtAyurojSB5di3MCd4deZsgS3KZK/edit?usp=drive_web
  // 
  // then the ID is the string of numbers in the middle
  //  
  //    1rACPT_W6E2u9taMKEIf2IijP6onTUtAyurojSB5di3MCd4deZsgS3KZK
  // 
  
  // Here is the Google Sheets that has all your questions.
  var question_bank_ID = '11OBP6CFOtgzgffGTk_GksHHVzGUSwZ7TwAY1nbMD5qE'
  
  // Here is the Google Sheets that you edit each time you want to select
  // different question numbers from the question bank. 
  var questions_for_quiz_ID = '1z9OrCqXHndRZwKvJI7bOEqbHavhVvj4B-E4981-C5b8'
  
  // Here is the Google Forms Quiz template that will get copied over each
  // time you want to make a new quiz. 
  var quiz_template_ID = '1757xgFGp_K_e9YICMeJm6Gpu9wHBPCvmr-FO54JakQY'
  
  /////////////////////////////////////////////////////////////
  
  ////////////////////////////////////////////////////////////
  // First, get values from "Create a quiz"
  ////////////////////////////////////////////////////////////

  var sscq = SpreadsheetApp.openById(questions_for_quiz_ID);

  var sheetcq = sscq.getSheets()[0];
  var title = sheetcq.getSheetValues(1,2,1,1)[0][0];
  var description = sheetcq.getSheetValues(2,2,1,1)[0][0];
  var allquestions = sheetcq.getSheetValues(3,2,1,100)[0];
  
  var questions = new Array();
  
  for(var i=0;i<allquestions.length;i++) {
    if(allquestions[i]) {
      questions.push(Math.floor(allquestions[i]));
    }
  }
  
  Logger.log(title);
  Logger.log(description);
  Logger.log(questions);

  // OK! We got all the relevant information

  ////////////////////////////////////////////////////////////  
  // Next, make a copy of our master quiz form template --- be sure to set it as a quiz to enble scoring 
  ////////////////////////////////////////////////////////////

  var master_quiz = DriveApp.getFileById(quiz_template_ID);
  
   var quiz = master_quiz.makeCopy(title);
   var id = quiz.getId();
    
   var form = FormApp.openById(id);
    
    form.setTitle(title)     
     .setDescription(description)
     .setConfirmationMessage('Thanks for responding!')
     ;
   


  ////////////////////////////////////////////////////////////
  // Let's get all the questions we want, and then add them to the 
  // new quiz!
  ////////////////////////////////////////////////////////////

  // Get the questions
  var ss = SpreadsheetApp.openById(question_bank_ID);
  
  var sheet = ss.getSheets()[0];  
  
  for (var i=0;i<questions.length;i++) {
    // startRow, startColumn, numRows, numColumns
    var text = sheet.getSheetValues(questions[i]+1, 2, 1, 1)[0][0];
    var qtype = sheet.getSheetValues(questions[i]+1, 3, 1, 1)[0][0];
    var desc = sheet.getSheetValues(questions[i]+1, 4, 1, 1)[0][0];
    var options = sheet.getSheetValues(questions[i]+1, 5, 1, 2)[0];
    var range = sheet.getRange(questions[i]+1, 5, 1, 2);
    var colors = range.getBackgrounds()[0];//this is a 2 dimentional array, so pick the first row only 
    var correctness = colors.map((x)=>x=='#ffffff'?false:true)
    
    Logger.log(colors);

    if(qtype=='MTF'){ // Multiple choice
      //form.addMultipleChoiceItem()
      var item = form.addCheckboxItem()
       
     // item.setHelpText(desc) // desc == explanation not needed right now 
      
      var true_checkboxes = [item.createChoice('True',true),item.createChoice('False',false)]
      var false_checkboxes = [item.createChoice('True',false),item.createChoice('False',true)]
      
      //need a make a loop that takes the value and correctness and create the choises 
        for (var k=0;k<options.length;k++) {
          item.setTitle(options[k]) // text == question
          if (correctness[k]==true){
            item.setChoices(true_checkboxes);
            item.setPoints(10);
          }
          else {
            item.setChoices(false_checkboxes);
            item.setPoints(10);
          }
        
        }
      
      //item.setChoices(checkboxes);
      //item.setPoints(10); 
    }
    else if(qtype=='SBA'){ // Checkbox
      var item = form.addCheckboxItem();
      item.setTitle(text);
      item.setHelpText(desc);
      var checkboxes = []
      
      //need a make a loop that takes the value and correctness and create the choises 
        for (var k=0;k<options.length;k++) {
        checkboxes.push(item.createChoice(options[k],correctness[k]))
        }
      
      item.setChoices(checkboxes);
      item.setPoints(10)
     //item.setFeedbackForCorrect('the reason is this')
     //item.setFeedbackForIncorrect('the reason not is this');
    }
    else if(qtype=='IU'){ // Image by URL
      var img = UrlFetchApp.fetch(text);
      var item = form.addImageItem()
      .setTitle(desc)
      .setHelpText(desc)
      .setImage(img)
      .setWidth(300); // Hard code the image width for now. 
    }
    else if(qtype=='ID'){ // Image from Drive, looking for ID
      var img = DriveApp.getFileById(text).getBlob();
      Logger.log(text);
      Logger.log(img);
      var item = form.addImageItem()
      .setTitle(desc)
      .setHelpText(desc)
      .setImage(img)
      .setWidth(300); // Hard code the image width for now. 
    }
    
    
  }
}
```
If you've added your own file IDs, you can run this script and it will make you a new Google Forms Quiz!

You'll need to add your own answer key and feedback, but this will allow you to reuse questions from one year to the next. 

TODO:
- [] add feedback with explanation
- [] add dynamic list sizing
- [] add optional images
- [] add answer shortcodes in testbank inputs 
- [] add keywords tagging in testbank inputs

