---
title: Sorting Quiz Questions
layout: default
description: Our Frontend talking to Backend Python application serving questions.  This api allows us to get customer responses. 
permalink: /data/quiz/sort
hide: true
image: /images/feedback.jpeg
tags: [javascript, fetch, dom, getElementID, appendChild]
---


{% include custom-head.html %}
<table id="flaskTable" class="table cell-border stripe" style="width:100%;">
    <thead id="flaskHead">
        <tr>
            <th>Subject</th>
            <th>ID</th>
            <th>Question</th>
            <th>Answer</th>
            <th>Pinned</th>
        </tr>
    </thead>
    <tbody id="flaskBody"></tbody>
</table>
 
<!-- Script is layed out in a sequence (without a function) and will execute when page is loaded -->
<script>
  $(document).ready(function() {
  
  dataX = {};
  fetch('http://localhost:5000/api/quiz/questions', { mode: 'cors' })
    .then(response => {
      if (!response.ok) {
        throw new Error('API response failed');
      }
      return response.json();
    })
    .then(data => {
      for (const row of data) {        
        dataX[row.id] = row;        
        $('#flaskBody').append('<tr><td>' + 
            row.subject + '</td><td>' + 
            row.qid + '</td><td>' + 
            row.question + '</td><td>' + 
            row.answer +  '</td>' +
            createPinnedColumn(row.id, row.pinned)
            + '</tr>'
        );        
      } 
      $("#flaskTable").DataTable();
      const pinb = document.getElementById('pinner');
      if (pinb != undefined) {
        pinb.addEventListener("click", handlePinEvent);
      }
    })
    .catch(error => {
      console.error('Error:', error);
    });
  });

  function createPinnedColumn(id, isPinned) {    
    label = '' ; // isPinned ? 'pinned' : 'unpinned';
    checked = (isPinned >= 1) ? 'checked' : '';
    b = '<td><input id="' + id + '" ' + isPinned +  
        '" type="checkbox" name="pinner" value="' + 
        label + '" ' + checked +
        ' onclick="handlePinEvent(event)">' + '<span style="margin-left:5px;">' + label + '</span>' + 
        '</td>';    
    return b;
  }
  function handlePinEvent(event) {  
    
    pinned = event.target.checked;
    rec = dataX[event.target.id];
    // TODO: uset setPinned api
    // event.target.value = pinned ? 'pinned' : 'unpinned';
      
  }

</script>