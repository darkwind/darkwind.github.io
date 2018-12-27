---
title: "TLV Parser for node.js"
date: 2018-12-27 13:32:00
layout: post
categories: [javascript]
tags: [nodejs, tlv]
---

TLV 파싱을 위한 클래스 개발
===

``` js
const moment = require('moment');
const { crc16xmodem } = require('crc');

class Parser {
  constructor() {
    this.buffer = Buffer.from([]);
  }

  // 시간을 버퍼에 넣기 4byte
  getBufferFromDate(d) {
    // UINT32 Big Endien 4byte
    const t = Buffer.from(Math.floor(d.getTime() / 1000).toString(16), 'hex');
    return t;
  }

  // 버퍼에서 시간 가져오기
  getDateFromBuffer(buf) {
    return moment(parseInt(buf.toString('hex'), 16) * 1000).format('YYYY-MM-DD HH:mm:ss');
  }

  encodeTLV(tag, value, valueType = null) {
    let t = Buffer.from([tag]);
    let l = Buffer.alloc(1);

    switch (valueType) {
    ┆ case 'int':
        // value 길이에 따라 몇 바이트로 쓸건지 결정
        if (value <= 255) {
          l.writeUInt8BE(v); 
        } else if (value <= 65535) {
          l.writeUInt16BE(v);  
        } else {
          console.error('[PARSER][ERROR] Overflow value');
          l.writeUInt16BE(v);   
        }
        break;
    ┆ case 'string':
        l.writeUInt8BE(v);
    ┆   break;
    ┆ default:
    ┆   type = Buffer.from([t]);
    ┆   l.writeUInt8(v.length);
    ┆   break;
    }

    return Buffer.concat([type, l, v]);
  }
}

```