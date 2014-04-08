# -*- coding: utf-8 -*-
# this file is released under public domain and you can use without limitations

#########################################################################
## This is our List Interpreter Final Project
## By: Dylan Markowitz, Edward Kuang, Sidhant Karamchandani, and Ryan Vacek
## Version: 12/8/13
## - index is the default action of any application
## - user is required for authentication and authorization
## - download is for downloading files uploaded in the db (does streaming)
## - call exposes all registered services (none by default)
#########################################################################

from __future__ import division

Symbol = str

@auth.requires_login()
def index():
    return dict(message=T('Welcome to our Lisp Interpreter!'),)

@auth.requires_login()
def index_ajax():
    interpret_url = URL('interpret')
    save_url = URL('save')
    loadedProject = ""
    my_record = db.project(db.project.id == request.args(0))
    if my_record:
        loadedProject = my_record.code
    #session.flash = "" + loadedProject
    return dict(interpret_url=interpret_url, save_url=save_url, loadedProject=loadedProject)

def tutorial():
    return dict(message=T('Lisp Tutorial'))

@auth.requires_login()
def interpret():
    s = request.vars.msg or ''
    #session.flash = "" + s
    s = s.replace('\\n', '~~~')    #replaces new lines with triple tilda's
    s = s.replace('\\', '')        #replaces \ with empty
    newStr = list(s)
    if newStr[0] == '"':           #gets rid of quotes at the front of the string
        newStr[0] = ''
    if newStr[-1] == '"':          #gets rid of quotes at the end of the string
        newStr[-1] = ''
    tempStr = "".join(newStr)
    listOfStr = list(paren_matcher(tempStr, '(', ')'))
    #session.flash += "" + str(listOfStr) + ": "
    evalList = []
    for x in listOfStr:
        if x != '~~~':
            evalList.append(eval2(parseString(str(x))))
            #session.flash += "@"
    #session.flash = "" + str(evalList)
    returnList = []
    for q in evalList:
        returnList.append(str(q))
    #session.flash = "" + returnVal
    return response.json(dict(result=returnList))

def paren_matcher(string, opens, closes):          #We borrowed this code from User torek on stackoverflow
    """Yield (in order) the parts of a string that are contained
    in matching parentheses.  That is, upon encounting an "open
    parenthesis" character (one in <opens>), we require a
    corresponding "close parenthesis" character (the corresponding
    one from <closes>) to close it.

    If there are embedded <open>s they increment the count and
    also require corresponding <close>s.  If an <open> is closed
    by the wrong <close>, we raise a ValueError.
    """
    stack = []
    if len(opens) != len(closes):
        raise TypeError("opens and closes must have the same length")
    # could make sure that no closes[i] is present in opens, but
    # won't bother here...

    result = []
    for char in string:
        # If it's an open parenthesis, push corresponding closer onto stack.
        pos = opens.find(char)
        if pos >= 0:
            if result and not stack: # yield accumulated pre-paren stuff
               yield ''.join(result)
               result = []
            result.append(char)
            stack.append(closes[pos])
            continue
        result.append(char)
        # If it's a close parenthesis, match it up.
        pos = closes.find(char)
        if pos >= 0:
            if not stack or stack[-1] != char:
                raise ValueError("unbalanced parentheses: %s" %
                    ''.join(result))
            stack.pop()
            if not stack: # final paren closed
                yield ''.join(result)
                result = []
    if stack:
        raise ValueError("unclosed parentheses: %s" % ''.join(result))
    if result:
        yield ''.join(result)

def parseString(s):                                #We borrowed this code from Peter Norvig's website on Lisp Interpreters
    "Read a Scheme expression from a string."
    return read_from(tokenize(s))

def tokenize(s):                                   #We borrowed this code from Peter Norvig's website on Lisp Interpreters
    "Convert a string into a list of tokens."
    return s.replace('(',' ( ').replace(')',' ) ').split()

def read_from(tokens):                             #We borrowed this code from Peter Norvig's website on Lisp Interpreters
    "Read an expression from a sequence of tokens."
    #session.flash += "" + str(tokens) + ":::"
    if len(tokens) == 0:
        raise SyntaxError('unexpected EOF while reading')
    token = tokens.pop(0)
    if '(' == token:
        L = []
        while tokens[0] != ')':
            L.append(read_from(tokens))
        tokens.pop(0) # pop off ')'
        return L
    elif ')' == token:
        raise SyntaxError('unexpected )')
    else:
        return atom(token)
    return "boo"

def atom(token):                                   #We borrowed this code from Peter Norvig's website on Lisp Interpreters
    "Numbers become numbers; every other token is a symbol."
    try:
        return int(token)
    except ValueError:
        try:
            return float(token)
        except ValueError:
            return Symbol(token)

class Env(dict):                                   #We borrowed this code from Peter Norvig's website on Lisp Interpreters
    "An environment: a dict of {'var':val} pairs, with an outer Env."
    def __init__(self, parms=(), args=(), outer=None):
        self.update(zip(parms,args))
        self.outer = outer
    def find(self, var):
        "Find the innermost Env where var appears."
        return self if var in self else self.outer.find(var)

def add_globals(env):                              #We borrowed this code from Peter Norvig's website on Lisp Interpreters
    "Add some Scheme standard procedures to an environment."
    import math, operator as op
    env.update(vars(math)) # sin, sqrt, ...
    env.update(
     {'!!':op.add, '-':op.sub, '*':op.mul, '/':op.div, 'not':op.not_,
      '>':op.gt, '<':op.lt, '>=':op.ge, '<=':op.le, '=':op.eq, 
      'equal?':op.eq, 'eq?':op.is_, 'length':len, 'cons':lambda x,y:[x]+y,
      'car':lambda x:x[0],'cdr':lambda x:x[1:], 'append':op.add,  
      'list':lambda *x:list(x), 'list?': lambda x:isa(x,list), 
      'null?':lambda x:x==[], 'symbol?':lambda x: isa(x, Symbol)})
    return env

global_env = add_globals(Env())

def eval2(x, env=global_env):                      #We borrowed this code from Peter Norvig's website on Lisp Interpreters
    "Evaluate an expression in an environment."
    if isa(x, Symbol):             # variable reference
        return env.find(x)[x]
    elif not isa(x, list):         # constant literal
        return x
    elif x[0] == 'quote':          # (quote exp)
        (_, exp) = x
        return exp
    elif x[0] == 'if':             # (if test conseq alt)
        (_, test, conseq, alt) = x
        return eval2((conseq if eval2(test, env) else alt), env)
    elif x[0] == 'set!':           # (set! var exp)
        (_, var, exp) = x
        env.find(var)[var] = eval2(exp, env)
    elif x[0] == 'define':         # (define var exp)
        (_, var, exp) = x
        env[var] = eval2(exp, env)
    elif x[0] == 'lambda':         # (lambda (var*) exp)
        (_, vars, exp) = x
        return lambda *args: eval2(exp, Env(vars, args, env))
    elif x[0] == 'begin':          # (begin exp*)
        for exp in x[1:]:
            val = eval2(exp, env)
        return val
    else:                          # (proc exp*)
        exps = [eval2(exp, env) for exp in x]
        proc = exps.pop(0)
        return proc(*exps)

isa = isinstance

@auth.requires_login()
def your_projects():
    q = db.project.designation == auth.user.email   #We select who can see the projects, in this case, only projects sent to you and created by you (you are the designated)
    grid = SQLFORM.grid(q,
        searchable=True,
        fields=[db.project.title, db.project.author, db.project.designation],
        csv=False,
        details=False, create=False, editable=False, deletable=True,
        links=[
            dict(header=T('View the Project'), #allows the viewing of a file
                 body = lambda r: A('View', _class='btn', _href=URL('default', 'view_project', args=[r.id], user_signature=True))),

            dict(header=T('Edit the Designation'), #allows the editing of a file
                 body = lambda r: A('Edit Designation', _class='btn', _href=URL('default', 'edit', args=[r.id], user_signature=True))),

            dict(header=T('Load the Code'), #allows the loading of a file
                 body = lambda r: A('Load', _class='btn', _href=URL('default', 'index_ajax', args=[r.id], user_signature=True))),
              ],
        )
    return dict(grid=grid)

@auth.requires_login()
def designated_projects():
    #Helper for query: i am the author and i am not the designated (projects you have sent)
    q = (db.project.author == auth.user.email) & (db.project.designation != auth.user.email)
    grid = SQLFORM.grid(q,
        searchable=True,
        fields=[db.project.title, db.project.author, db.project.designation],
        csv=False,
        details=False, create=False, editable=False, deletable=False,
        links=[
            dict(header=T('View the Project'), #allows the viewing of a file
                 body = lambda r: A('View', _class='btn', _href=URL('default', 'view_project', args=[r.id], user_signature=True)),
            )],
        )
    return dict(grid=grid)

@auth.requires_signature()
def view_project(): #shows the information stored in a project
    my_record = request.args(0)
    if my_record is None:
        session.flash = "Invalid request"
        redirect(URL('default', 'index'))
    form = SQLFORM(db.project, record=my_record, readonly=True)
    return dict(form=form)

@auth.requires_signature()
def edit():
    my_record = db.project(request.args(0))
    if my_record is None:
        session.flash = "Invalid request"
        redirect(URL('default', 'index'))
    #only designation should be editable here:
    db.project.author.writable = False
    db.project.title.writable = False
    db.project.code.writable = False
    db.project.designation.writable = True
    form = SQLFORM(db.project, record=my_record)
    if form.process().accepted:
        redirect(URL('default', 'your_projects'))
    return dict(form=form)

def save():
    oldStr= request.vars.msg or ''
    (tempTitle, tempInput) = oldStr.split(':::::')
    finalInput = tempInput.replace('\\n', '~~~')
    finalInput = finalInput.replace('\\', '') #cant have strings in our lisp interpreter
    finalInput = finalInput.replace('!!', '+')
    #session.flash = "" + tempInput
    db.project.insert(title=tempTitle, code=finalInput, designation=auth.user.email)
    return dict(message=T('It worked'))
    #return response.json(dict(result=s))

def user():
    """
    exposes:
    http://..../[app]/default/user/login
    http://..../[app]/default/user/logout
    http://..../[app]/default/user/register
    http://..../[app]/default/user/profile
    http://..../[app]/default/user/retrieve_password
    http://..../[app]/default/user/change_password
    http://..../[app]/default/user/manage_users (requires membership in
    use @auth.requires_login()
        @auth.requires_membership('group name')
        @auth.requires_permission('read','table name',record_id)
    to decorate functions that need access control
    """
    return dict(form=auth())

@cache.action()
def download():
    """
    allows downloading of uploaded files
    http://..../[app]/default/download/[filename]
    """
    return response.download(request, db)


def call():
    """
    exposes services. for example:
    http://..../[app]/default/call/jsonrpc
    decorate with @services.jsonrpc the functions to expose
    supports xml, json, xmlrpc, jsonrpc, amfrpc, rss, csv
    """
    return service()


@auth.requires_signature()
def data():
    """
    http://..../[app]/default/data/tables
    http://..../[app]/default/data/create/[table]
    http://..../[app]/default/data/read/[table]/[id]
    http://..../[app]/default/data/update/[table]/[id]
    http://..../[app]/default/data/delete/[table]/[id]
    http://..../[app]/default/data/select/[table]
    http://..../[app]/default/data/search/[table]
    but URLs must be signed, i.e. linked with
      A('table',_href=URL('data/tables',user_signature=True))
    or with the signed load operator
      LOAD('default','data.load',args='tables',ajax=True,user_signature=True)
    """
    return dict(form=crud())
