---
layout: post
comments: true
title:  "Bug in browser"
date:   2019-5-19 13:22:16 +0800
categories: jekyll update
img: buginbrowser.jpg # Add image post (optional)
tags: [bug, chrome, ie]
---

## Bug

Client complains a bug that the download function will fail on chrome browser consistently, 
while in same page same download function will success. While client change the browser to IE,
 then the two download functions will both be success.

By checking the log in backend, all the 4 combination operations return 200 success code in backend server 

When tracked to the source code of the two operations:

```scala
def downloadInvestors(fiId: String, year: Int) = deadbolt.Pattern(value = "v_pwc", patternType = PatternType.REGEX)() { 
  implicit request => {
    investorFiInvestmentServiceRead.listAsExcel(fiId, year).map(workbook => {
      val bstream = new ByteArrayOutputStream()
      workbook.write(bstream)
      Ok(bstream.toByteArray).withHeaders(CONTENT_TYPE -> "application/x-download",
       CONTENT_DISPOSITION -> s"attachment; filename=fiInvestors.xlsx")
    })
  }
}

def downloadInvestments(fiId: String, year: Int) = deadbolt.Pattern(value = "v_pwc", patternType = PatternType.REGEX)() { 
  implicit request => {
    fiuserServiceRead.lookup(fiId).flatMap(fo => {
      fo.map(fi => {
        fiCompanyInvestmentServiceRead.listAsExcel(fiId, year).map(workbook => {
          val bstream = new ByteArrayOutputStream()
          workbook.write(bstream)
          Ok(bstream.toByteArray).withHeaders(CONTENT_TYPE -> "application/x-download",
           CONTENT_DISPOSITION -> s"attachment; filename=FICompanyInvestments_${fi.name}_$year.xlsx")
        })
      }).getOrElse(Future(BadRequest))
    }).recover {
      case e: Exception => {
        Logger.logger.error(s"Download fiCompanyInvestments ${fiId}", e)
        InternalServerError
      }
    }
  }
}

```

The first function will be success, while the second function will be fail only in chrome. What’s the different of the two functions?

The only difference is that the second function use the API's parameters for the download’s file name. Then the bug happens: 

when there is some special characters in {fi.name} like “X, Y” then the chrome download will fail. Because the chrome browser
 applies more strictly secure rule about the name of save download file. The name pattern will not pass the chrome’s rule
  then even the file is download to local (the backend server returns 200) after that the chrome block the operation save 
  the file to that specified name, then download operation failed, when use the debug tool to monitor the API call, the
   operation marked failed on server.

## How to resolve the bug?

Just simple not trust any user’s input, and rename the download file in client side.

## How to prevent this kind of bug?

All download operation use same implementation, in the code about, the two functions have same operation while have different implementations.




[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
