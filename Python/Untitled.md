---
title: ""
date: 2021-01-22T17:27:21+08:00
draft: false
---
```python
import random
capitals={{'Alabama': 'Montgomery', 'Alaska': 'Juneau', 'Arizona': 'Phoenix','Arkansas': 'Little Rock', 'California': 'Sacramento', 
           'Colorado': 'Denver','Connecticut': 'Hartford', 'Delaware': 'Dover', 'Florida': 'Tallahassee','Georgia': 'Atlanta', 
           'Hawaii': 'Honolulu', 'Idaho': 'Boise', 'Illinois':'Springfield', 'Indiana': 'Indianapolis', 'Iowa': 'Des Moines', 
           'Kansas':'Topeka', 'Kentucky': 'Frankfort', 'Louisiana': 'Baton Rouge', 'Maine':'Augusta', 'Maryland': 'Annapolis', 
           'Massachusetts': 'Boston', 'Michigan':'Lansing', 'Minnesota': 'Saint Paul', 'Mississippi': 'Jackson', 'Missouri':'Jefferson City', 
           'Montana': 'Helena', 'Nebraska': 'Lincoln', 'Nevada':'Carson City', 'New Hampshire': 'Concord', 'New Jersey': 'Trenton', 
           'NewMexico': 'Santa Fe', 'New York': 'Albany', 'North Carolina': 'Raleigh','North Dakota': 'Bismarck', 'Ohio': 'Columbus', 
           'Oklahoma': 'Oklahoma City','Oregon': 'Salem', 'Pennsylvania': 'Harrisburg', 'Rhode Island': 'Providence','South Carolina': 'Columbia', 
           'South Dakota': 'Pierre', 'Tennessee':'Nashville', 'Texas': 'Austin', 'Utah': 'Salt Lake City', 'Vermont':'Montpelier',
           'Virginia': 'Richmond', 'Washington': 'Olympia', 'WestVirginia': 'Charleston', 'Wisconsin': 'Madison', 'Wyoming': 'Cheyenne'}
for quizNum in range(35):
           quizFile=open('capitalsquiz%s.txt'%(quizNum+1),'w')  # %s会被(quizNum+1)替代
           answerkeyFile=open('capitalsquiz_answers%s.txt'%(quizNum+1),'w')
           quizFile.write('Name:\n\nDate:\n\nPeriod:\n\n')
           quizFile.write((' '*20)+'State Capitals Quiz (From %s)'%(quizNum+1))
           quizFile.write('\n\n')
           states=list(capitals.keys())
           random.shuffle(states)  # 重新随机排列该列表中的值
           for questionNum in range(50):
                   correctAnswer=capitals[states[questionNum]]
                   wrongAnswers=list(capitals.values())
                   del wrongAnswers[wrongAnswers.index(correctAnswer)]
                   wrongAnswers=random.sample(wrongAnswers,3)
                   answerOptions=wrongAnswers+[correctAnswer]
                   random.shuffle(answerOptions)
                   quizFile.write('%s. what is the capital of %s\n'%(questionNum+1,states[questionNum]))
                   for i in range(4):
                           quizFile.write(' %s. %s\n'%('ABCD'[i],answerOption[i]))
                           quizFile.write('\n')
                           answerKeyFile.write('%s. %s\n'%(questionNum+1,'ABCD'[answerOptions.index(correctAnswer)))
                   quizFile.close()
                   answerKeyFile.close()
```


      File "<ipython-input-1-908afecd34e2>", line 12
        for quizNum in range(35):
                                ^
    SyntaxError: invalid syntax


