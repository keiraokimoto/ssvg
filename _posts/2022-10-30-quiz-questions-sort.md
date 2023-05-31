---
title: Sorting Quiz Questions
layout: base
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
            <th id="c1">Subject</th>
            <th id="c2">ID</th>
            <th>Question</th>
            <th>Answer</th>
            <th>Pinned</th>
            <th style="display:none"></th>
        </tr>
    </thead>
    <tbody id="flaskBody"></tbody>
</table>
 
<!-- Script is layed out in a sequence (without a function) and will execute when page is loaded -->
<script>
  const options = {
    method: 'GET', // *GET, POST, PUT, DELETE, etc.
    mode: 'cors', // no-cors, *cors, same-origin
    cache: 'default', // *default, no-cache, reload, force-cache, only-if-cached
    credentials: 'omit', // include, *same-origin, omit
    headers: {
      'Content-Type': 'application/json'
      // 'Content-Type': 'application/x-www-form-urlencoded',
    },
  };
  // prepare fetch PUT options, clones with JS Spread Operator (...)
  const put_options = {...options, method: 'PUT'}; // clones and replaces method
  sorting=false;
  refreshNeeded = true;
  headerClicked = false;
  $(document).ready(function() {
  
    dataX = {};
    fetchData();

  });

  function createPinnedColumn(id, isPinned) {    
    label = '' ; // isPinned ? 'pinned' : 'unpinned';
    checked = (isPinned >= 1) ? 'checked' : '';
    b = '<td><input id="' + id + '" ' +
        ' type="checkbox" name="pinner"  ' + checked +
        ' onclick="handlePinEvent(event)">' + '<span style="margin-left:5px;">' + label + '</span>' + 
        '</td>';    
    return b;
  }

  function handlePinEvent(event) {  
    
    pinned = event.target.checked;
    rec = dataX[event.target.id];
    savePin(event.target.id, pinned);
    
  }

  function savePin(qno, pinned ) {

      pinSelected = true;
      p = pinned ? 1 : 0;
      dataX[qno].pinned = p;
      refreshNeeded = false;

      extrapath = "/" + qno + "/" + p;
      url = 'http://localhost:5000/api/quiz/questions/pin' + extrapath;
      fetch(url, put_options)
        .then(response => {
          if (!response.ok) {
            throw new Error('API response failed');
          }
          return response.json();
        })
        .then(data => {
          refreshNeeded =  false;
          pinSelected = true;
          removeTable();
          fetchData();
        })
        .catch(error => {
          console.error('Error:', error);
        });
  }

  function fetchData() {
      fetch('http://localhost:5000/api/quiz/questions', { mode: 'cors' })
        .then(response => {
          if (!response.ok) {
            throw new Error('API response failed');
          }
          return response.json();
        })
        .then(data => {
          initTable(data);
          refreshNeeded = true;
        })
        .catch(error => {
          console.error('Error:', error);
        });
  }

  function initTable(data) {

    
    for (const row of data) {        
      dataX[row.id] = row;        
      $('#flaskBody').append(formatrow(row));        
    } 
     tbl = //$("#flaskTable").DataTable();
    $('#flaskTable').DataTable({
      'columnDefs': [
      {
          'targets': [5],
          'visible': false,
          'searchable': false
      },
      {
          'targets': [4], 
          'orderable': false, // set orderable false for selected columns
      }
]     ,
    });
         
    $('#flaskTable thead').on( 'click', 'th', function () {         
            sorting = true;
            pinSelected = false;
            headerClicked = true;
    } );

  tbl.on('order.dt', function () {
      sorting =  true;
  });
   
   $('#flaskTable').on( 'draw.dt', function () {
      console.log( 'Table redrawn' );
      if (sorting && refreshNeeded) {
        reorderRows();
        sorting = false;
      }
    });
     
  }
  
  function formatrow(row) {
    var f = '<tr id="' + row.id + '" ><td>' + 
          row.subject + '</td><td>' + 
          row.qid + '</td><td>' + 
          row.question + '</td><td>' + 
          row.answer +  '</td>' +
          createPinnedColumn(row.id, row.pinned) +
          '<td>' + row.pinned + '</td>' +
          '</tr>';
    return f;
    
  }
  
  function reorderRows() {

    pins = [];
    
    var tbl = $('#flaskTable').DataTable();

    refreshNeeded = false;
    orders = tbl.order();
    pinorders =[ [ 5, "desc" ]];
    pinorders.push(orders[0]);
    tbl.order(pinorders).draw();
/*
    tbl.rows( function ( idx, data, node ) {
      var r = dataX[node.id].pinned === 1;
      if (r) {
        pins.push(node);
      //  tbl.row(idx).remove();
      }
      return r;
    } ).remove().draw();

    pins.slice().reverse()
      .forEach(function(row) {
        $('#flaskBody').prepend(row);
    });
    */
    refreshNeeded = true;  
    sorting = false;
  }

  function removeTable() {
    var removeTab = document.getElementById('flaskBody');
    var parentEl = removeTab.parentElement;
    
    tbl = $("#flaskTable").DataTable();
    tbl.clear().draw();
    tbl.destroy();
    sorting = false;
  }

  
  function initData() {
    
    tbl = $('#flaskTable').DataTable();
    var firstRows = []; // new row in the datatable
    var lastRows = [];
    var keys = new Set();
    tbl.rows( function ( idx, data, node ) {
       var m = $(node).find('input[type="checkbox"]');
       d = dataX[node.id];
       if (!keys.has(node.id)) {
        keys.add(node.id);
        if (d.pinned) {
          firstRows.push(d);
        }
        else {
          lastRows.push(d);
        }
       }
    });
    l = firstRows.concat(lastRows);
    removeTable();
    initTable(l);     
  }

  
</script>