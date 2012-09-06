fs = require 'fs'
xml2js = require 'xml2js'
util = require 'util'

{ Deferred } = require 'simply-deferred'

option '-i', '--input [PATH]', 'path for svg'

deferredCallback = (deferred) ->
  (err, result) ->
    if err
      return deferred.reject(err)
    deferred.resolve(result)

promise = (fn) ->
  deferred = new Deferred()
  ->
    fn(deferredCallback(deferred))
      .apply(this, arguments)
    return deferred.promise()

svg2json = promise (callback) ->
  (svg) ->
    parser = new xml2js.Parser()
    parser.parseString(svg, callback)

readFile = promise (callback) ->
  (filename) ->
    fs.readFile(filename, callback)

toString = (buffer) -> buffer.toString()

tokens = [
  /^([Mm])([0-9-,.]+)/ # moveto
  /^([CcSs])([0-9-,.]+)/ # curveto
  /^([Ll])([0-9-,.]+)/ # lineto
]

parse = (path) ->
  i = 0
  vertices = []
  absolute = [0, 0]

  while chunk = path[i..]
    for token in tokens
      unless match = token.exec chunk
        continue

      [text, type, data] = match
      i += text.length - 1

      data = data.replace(/\-/g, ',-')
        .replace(/^,/, '')

      points = data.split ','

      while points.length
        x = parseFloat(points.shift())
        y = parseFloat(points.shift())

        if /[A-Z]/.test(type)
          vertex = absolute = [x, y]
        else
          vertex = [x + absolute[0], y + absolute[1]]

        vertices.push(vertex)
      break

    i++

  return vertices

task 'svg2json', 'parse an svg file into json', (options) ->
  readFile(options.input).pipe(toString).done (svg) ->
    console.log svg
    svg2json(svg).done (json) ->
      paths = []

      for shapeType, shapes of json
        unless shapeType is 'path'
          continue

        unless Array.isArray(shapes)
          paths.push(parse(shapes['@'].d))
          return

        for shape in shapes
          paths.push(parse(shape['@'].d))

      console.log JSON.stringify(paths: paths, null, 2)
